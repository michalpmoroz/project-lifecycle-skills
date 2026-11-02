# Backend API Runtime & Database Performance Checklist (PostgreSQL 17+)

> **Baseline Runtimes & Database Versions**:
> - **Backend Runtimes**: Node.js 22 LTS (V8 12.4+), Bun 1.2+, Go 1.23+, Python 3.13+ (`uv` / `asyncio`)
> - **Relational & Embedded Databases**: PostgreSQL 17.x, PgBouncer 1.23+ / Supavisor, SQLite 3.47+ (WAL mode)
> - **Message & Job Queues**: BullMQ 5.x (Redis 7.4+ / Valkey 8.0+), `pg-boss` (PostgreSQL), NATS JetStream / RabbitMQ
> - **API Latency Targets**: Read Endpoints P95 ≤ 100ms, Write/Mutation Endpoints P95 ≤ 250ms, Event Loop Lag P99 < 20ms

---

## 1. Backend Runtime & API Layer Optimization

### 1.1 Eliminating N+1 Query Loops & DataLoader Batching
- **Rule**: Never execute a database query or downstream HTTP/gRPC call inside a `for` / `.map()` loop over a collection of parent entities.
- **Multi-Domain Impact**:
  - *E-commerce*: Fetching 50 orders and running 50 separate queries to load each order's line items (`1 + 50` queries).
  - *Healthcare*: Fetching 40 appointments and querying each doctor's specialty individually.
  - *Civic / GovTech*: Fetching 100 electoral districts and querying each district's incumbent representative in a loop.
- **Remedies**:
  1. **Relational `JOIN` or `WHERE id = ANY($1::uuid[])`**: Fetch all child records in a single round-trip and group them in memory by foreign key.
  2. **GraphQL / Nested Resolver `DataLoader`**: Use request-scoped tick batching (`dataloader`) to coalesce all `.load(id)` calls occurring within the same event-loop tick into a single batch query.

```typescript
// ANTI-PATTERN: N+1 query inside loop (51 DB round-trips for 50 orders)
// for (const order of orders) {
//   order.items = await db.query('SELECT * FROM order_items WHERE order_id = $1', [order.id]);
// }

// BEST PRACTICE: Single parameterized ANY($1) batch query (2 DB round-trips total)
const orderIds = orders.map((o) => o.id);
const allItems = await db.query<OrderItem>(
  'SELECT id, order_id, sku, quantity, unit_price_cents FROM order_items WHERE order_id = ANY($1::uuid[])',
  [orderIds]
);
const itemsByOrderId = Map.groupBy(allItems.rows, (item) => item.order_id);
for (const order of orders) {
  order.items = itemsByOrderId.get(order.id) ?? [];
}
```

### 1.2 Database Connection Pooling & Serverless Pool Exhaustion
- **Rule**: Never open a raw TCP+TLS database connection per HTTP request.
  - In long-running containers (Node.js/Go/Python), configure a bounded connection pool sized according to the PostgreSQL formula: `connections = ((core_count * 2) + effective_spindle_count)` divided across replica instances.
  - In serverless/edge functions or high-replica Kubernetes deployments, always route through an external transaction-mode connection pooler (**PgBouncer** or **Supavisor**) so 1,000 concurrent lambda/worker invocations multiplex over 20–50 physical PostgreSQL backend processes.
- **Statement Timeout Mandate**: Always configure `statement_timeout = '5000ms'` and `idle_in_transaction_session_timeout = '10000ms'` so a single runaway query or crashed client transaction cannot hold row locks or starve the connection pool.

### 1.3 Streaming Large Responses (NDJSON / Streams / SSE) vs. Memory Buffering
- **Rule**: Never buffer multi-megabyte datasets (e.g., 100,000-row financial ledger exports, healthcare FHIR bulk exports, or large AI/LLM completions) into a single in-memory array before calling `JSON.stringify()`.
- **Why**: Buffering 100MB per request spikes V8/Python heap memory, triggers stop-the-world Garbage Collection pauses across all concurrent requests, and risks `OOMKilled` container crashes.
- **Pattern**: Stream database cursors directly through a `TransformStream` or Node.js `pipeline` as **NDJSON (`application/x-ndjson`)**, CSV chunks, or **Server-Sent Events (`text/event-stream`)** with proper backpressure handling.

```typescript
import { pipeline } from 'node:stream/promises';
import { Transform } from 'node:stream';

// Streaming DB cursor rows as NDJSON with constant ~5MB RAM footprint regardless of table size
export async function streamAuditExport(dbClient: PoolClient, res: ServerResponse): Promise<void> {
  res.writeHead(200, {
    'Content-Type': 'application/x-ndjson; charset=utf-8',
    'Transfer-Encoding': 'chunked',
    'X-Content-Type-Options': 'nosniff',
  });

  const cursorStream = dbClient.query(new Cursor('SELECT id, actor_id, action, created_at FROM audit_events'));
  const toNdjson = new Transform({
    objectMode: true,
    transform(row, _encoding, callback) {
      callback(null, `${JSON.stringify(row)}\n`);
    },
  });

  await pipeline(cursorStream, toNdjson, res);
}
```

### 1.4 Node.js / Event Loop Health & CPU Blocking Prevention
- **Rule**: Keep P99 Event Loop lag **< 20ms** (monitor via `perf_hooks.monitorEventLoopDelay()`).
- **Strict Prohibitions on Hot Request Paths**:
  1. **Zero Synchronous I/O**: Never call `fs.readFileSync`, `crypto.pbkdf2Sync`, `bcrypt.hashSync`, or `child_process.execSync` after server startup.
  2. **ReDoS (Regular Expression Denial of Service)**: Never run unbounded backtracking regular expressions (`/(a+)+$/`) against untrusted user input. Use linear-time regex engines (`re2`) or strict input length caps before regex evaluation.
  3. **JSON Serialization Bottlenecks**: Project only required columns in SQL (`SELECT id, title, slug` instead of `SELECT *` containing 50KB `TEXT`/`JSONB` blobs) and use schema-compiled serializers (e.g., `fast-json-stringify` / TypeBox) on high-throughput endpoints.

### 1.5 HTTP Response Compression Thresholds & Content-Type Filtering
- **Rule**: When configuring dynamic response compression (`Brotli`, `Zstandard`, `Gzip`) at the API gateway or application server:
  1. **Enforce a Minimum Size Threshold (`>= 1024` bytes / 1 KB)**: Never compress tiny JSON responses (`< 1 KB`, such as `{"status":"ok"}` or single-record health/status checks). Below 1 KB, payload bytes already fit inside a single TCP MTU packet (`~1,460 bytes`), and running Brotli/Gzip wastes CPU while sometimes inflating byte size due to compression header/dictionary framing.
  2. **Exclude Pre-Compressed Binary MIME Types**: Strictly restrict dynamic compression to compressible text/structured formats (`application/json`, `text/html`, `text/plain`, `application/x-ndjson`, `image/svg+xml`). Never waste CPU attempting to re-compress `image/avif`, `image/webp`, `image/jpeg`, `video/*`, `application/zip`, or `application/pdf`.

### 1.6 Asynchronous Worker Queues for Heavy Compute
- **Rule**: Any operation requiring **> 100ms of CPU time** or **unpredictable third-party network I/O** must be decoupled from the synchronous HTTP request/response cycle and dispatched to a durable background job queue (e.g., BullMQ on Redis/Valkey, `pg-boss` on PostgreSQL, or RabbitMQ/NATS):
  - Transactional email / SMS / push notifications.
  - PDF invoice / medical report generation.
  - Image/video resizing and AVIF/WebP transcoding.
  - Bulk CSV/XLSX ingestion and webhook fan-out.
- **API Pattern**: Return `202 Accepted` immediately with a job status resource URL (`/api/v1/jobs/{jobId}`) or push completion via SSE/WebSockets.

---

## 2. Database & Storage Optimization (PostgreSQL 17+)

### 2.1 Precision Indexing Strategy (B-Tree, Composite, Covering, Partial, GIN)
Every frequent `WHERE`, `JOIN`, and `ORDER BY` clause must be backed by the right index type:

1. **Composite B-Tree Column Ordering (Equality Before Range)**:
   - Place **equality (`=`)** columns first, and **range/sort (`>`, `<`, `BETWEEN`, `ORDER BY`)** columns second.
   - *Example*: For `WHERE tenant_id = $1 AND status = 'ACTIVE' ORDER BY created_at DESC`, the optimal index is `(tenant_id, status, created_at DESC)`.
2. **Covering Indexes (`INCLUDE`) for Index-Only Scans**:
   - Append non-filter payload columns via `INCLUDE` so PostgreSQL can answer the query entirely from the index pages without touching the table heap (`Index Only Scan` with `Heap Fetches: 0`).
3. **Partial Indexes (`WHERE` clause on index)**:
   - Index only the hot subset of rows. If 98% of orders in an E-commerce database are `'COMPLETED'` and background workers only poll `'PENDING'` orders, create a partial index that is 50x smaller in RAM.
4. **GIN Indexes for `JSONB`, Arrays, and Full-Text Search (`tsvector`)**:
   - Use `USING gin (metadata jsonb_path_ops)` for fast `@>` containment queries on JSONB columns.
5. **Unindexed Foreign Keys Warning**:
   - PostgreSQL automatically creates an index on Primary Keys, **not** on Foreign Keys. Always create an explicit B-Tree index on every Foreign Key column to prevent full table scans during `JOIN`s and parent `ON DELETE CASCADE` checks.

```sql
-- 1. Covering Index enabling Index-Only Scan for user feed lookups
CREATE INDEX CONCURRENTLY idx_posts_author_created_cover
  ON posts (author_id, created_at DESC)
  INCLUDE (id, title, summary_slug);

-- 2. Partial Index indexing only active/pending jobs (tiny memory footprint)
CREATE INDEX CONCURRENTLY idx_jobs_pending_queue
  ON background_jobs (priority DESC, scheduled_at ASC)
  WHERE status = 'PENDING';

-- 3. GIN jsonb_path_ops index for high-speed JSONB containment (@>) queries
CREATE INDEX CONCURRENTLY idx_products_attributes_gin
  ON products USING gin (attributes jsonb_path_ops);
```

### 2.2 Mandatory Execution Plan Verification (`EXPLAIN (ANALYZE, BUFFERS)`)
- **Rule**: Never judge a query by its wall-clock time on an empty local dev database. Always verify production-scale queries using `EXPLAIN (ANALYZE, BUFFERS, SETTINGS)`.
- **What to Inspect**:
  - Watch `Buffers: shared hit=... read=...`: High buffer counts (> 1,000 pages = > 8MB of data scanned to return 20 rows) reveal missing or poorly selective indexes even if the data happened to be cached in RAM.
  - Watch for `Rows Removed by Filter: 95000` (indicates the index did not cover the filter predicate) or `external merge Disk:` (indicates `work_mem` spilled to disk during a large sort/hash).

### 2.3 Keyset / Cursor Pagination vs. Deep `OFFSET` Degradation
- **Rule**: **Prohibit `OFFSET` pagination** (`LIMIT 20 OFFSET 50000`) on any table capable of growing beyond 10,000 rows.
- **Why**: `OFFSET 50000` forces PostgreSQL to scan, fetch, and sort **50,020 rows**, discard the first 50,000 rows, and return 20—causing latency to degrade linearly ($O(N)$) with page depth.
- **Best Practice (Keyset / Seek Pagination)**: Use a deterministic compound cursor `(created_at, id)` backed by a matching composite index `(created_at DESC, id DESC)`, delivering constant **$O(1)$ sub-millisecond performance** on page 1 or page 100,000.

```sql
-- ANTI-PATTERN: Deep OFFSET scans and discards 100,000 rows (~450ms)
-- SELECT id, created_at, amount_cents FROM ledger_entries
-- ORDER BY created_at DESC, id DESC LIMIT 25 OFFSET 100000;

-- BEST PRACTICE: Keyset Seek Pagination using row-value comparison (< 0.8ms at any depth!)
SELECT id, created_at, amount_cents
FROM ledger_entries
WHERE (created_at, id) < ($1::timestamptz, $2::uuid)
ORDER BY created_at DESC, id DESC
LIMIT 25;
```

### 2.4 PostgreSQL HOT (Heap-Only Tuple) Updates & `fillfactor` Tuning
- **The Problem**: Under PostgreSQL's MVCC architecture, every `UPDATE` writes a new physical row version (tuple). If an table has 6 indexes and the new tuple does not fit on the same 8 KB heap page (or modifies any indexed column), PostgreSQL must update **all 6 indexes** on every single `UPDATE`—causing severe write amplification, WAL bloat, and `autovacuum` lag on write-heavy tables (e.g., E-commerce inventory stock counters, FinTech wallet balances, Civic live precinct vote tallies, SaaS worker heartbeats).
- **Rule**: For tables experiencing high-frequency `UPDATE` workloads:
  1. Lower the table `fillfactor` from `100` to **`80` or `85`** (`ALTER TABLE ... SET (fillfactor = 80)`) so every 8 KB page reserves 15%–20% free space for in-page **HOT (Heap-Only Tuple)** updates.
  2. Avoid indexing frequently mutated counter/heartbeat columns (`view_count`, `last_seen_at`, `updated_at`) unless strictly required by critical queries, so updates qualify for zero-index-write HOT chains.

```sql
-- Reserve 20% free space per 8KB heap page so frequent balance/inventory updates execute as fast HOT updates
ALTER TABLE inventory_stock SET (
  fillfactor = 80,
  autovacuum_vacuum_scale_factor = 0.02
);
```

### 2.5 Read Replicas, Materialized Views & Bulk Upserts
- **Materialized Views (`REFRESH MATERIALIZED VIEW CONCURRENTLY`)**: Precompute expensive multi-table analytical aggregations (e.g., Civic election turnout percentages by region, E-commerce daily sales leaderboards, Healthcare hospital bed occupancy) into a Materialized View with a unique index and refresh it asynchronously via cron/worker every 1–5 minutes.
- **Bulk Writes (`UNNEST` / `COPY`)**: When inserting or updating 1,000+ rows, never execute 1,000 individual `INSERT` statements. Use `INSERT INTO ... SELECT * FROM UNNEST($1::uuid[], $2::text[], $3::int[])` or PostgreSQL `COPY` to achieve 20x–50x higher write throughput.
