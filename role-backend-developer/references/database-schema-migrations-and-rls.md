# Database Schema Design, Zero-Downtime Migrations & RLS Checklist (`references/database-schema-migrations-and-rls.md`)

> **Baseline Versions**: PostgreSQL `17.x`, SQLite `3.46+` / LibSQL (Turso), Prisma `6.x`, Drizzle ORM `0.38+`, Kysely `0.27+`.
> **Mandate**: Load and enforce this checklist whenever designing database schemas, authoring DDL migrations, optimizing SQL queries, configuring connection pools, or implementing multi-tenant isolation.

> [!NOTE]
> **Continuous Evolution Mandate**: This checklist is a baseline floor, not a static ceiling. Engineers must proactively verify query execution plans (`EXPLAIN (ANALYZE, BUFFERS)`), lock contention, and data integrity invariants specific to the domain workload.

---

## 1. Normalization vs. Tactical Denormalization

- [ ] **3NF Default**: Model core transactional entities in Third Normal Form (3NF) with strict foreign keys and `CHECK` constraints.
- [ ] **Immutable Historical Snapshots**: When recording point-in-time legal or financial facts, intentionally denormalize snapshot data (or store an immutable JSONB snapshot validated by schema) so future master-record edits do not corrupt history:
  - *E-commerce / FinTech*: Copy unit price, tax rate, and billing address onto `invoice_line_items` at purchase time rather than joining mutable `products` or `addresses` tables.
  - *Healthcare*: Snapshot patient weight, allergies, and dosage guidelines on the `medication_order` record at the moment of clinical sign-off.
  - *Civic / GovTech*: Snapshot district boundaries and candidate metadata at election certification time.
- [ ] **PostgreSQL 17 Native Capabilities**: Leverage UUIDv7 (time-ordered UUIDs for B-Tree locality and zero page-split fragmentation), generated columns (`GENERATED ALWAYS AS ... STORED`), and `MERGE` statements where appropriate.

---

## 2. Zero-Downtime Expand-and-Contract Migrations

Never execute lock-heavy or backward-incompatible DDL against tables serving live application instances:

| Desired Change | Unsafe Anti-Pattern (Causes Downtime / Errors) | Safe Expand-and-Contract Protocol |
| :--- | :--- | :--- |
| **Rename a column** | `ALTER TABLE t RENAME COLUMN a TO b;` (Breaks running app pods expecting `a`) | 1. **Expand**: Add nullable column `b`, add trigger or app dual-write to both `a` and `b`.<br>2. **Backfill**: Batch-update existing rows (`WHERE b IS NULL LIMIT 5000`).<br>3. **Switch**: Deploy app reading/writing only `b`.<br>4. **Contract**: Drop column `a` in a subsequent release. |
| **Add `NOT NULL` constraint** | `ALTER TABLE t ADD COLUMN c text NOT NULL;` (without default) or `ALTER TABLE t ALTER COLUMN c SET NOT NULL;` (full table scan lock) | 1. Add column nullable.<br>2. Backfill in batches.<br>3. Add `ALTER TABLE t ADD CONSTRAINT chk_c_not_null CHECK (c IS NOT NULL) NOT VALID;`<br>4. Run `ALTER TABLE t VALIDATE CONSTRAINT chk_c_not_null;` (acquires only `SHARE UPDATE EXCLUSIVE` lock).<br>5. In PG 12+, `SET NOT NULL` can then skip the full scan automatically. |
| **Create an index** | `CREATE INDEX idx_x ON t (col);` (Blocks writes via `SHARE` lock) | Always use `CREATE INDEX CONCURRENTLY idx_x ON t (col);` outside a transaction block. Check `pg_stat_progress_create_index` and verify no `INVALID` indexes remain afterward. |
| **Add Foreign Key** | `ALTER TABLE child ADD CONSTRAINT fk ... REFERENCES parent(id);` (Locks both tables while validating) | 1. `ALTER TABLE child ADD CONSTRAINT fk ... NOT VALID;`<br>2. `ALTER TABLE child VALIDATE CONSTRAINT fk;` |

---

## 3. Transaction Isolation & Locking Strategies

- [ ] **Short Transactions**: Never perform external HTTP calls, file uploads, LLM calls, or slow email dispatches inside an open database transaction (`BEGIN ... COMMIT`).
- [ ] **Optimistic Concurrency Control (OCC)**: Include a `version integer NOT NULL DEFAULT 1` column on user-editable records. Update with `UPDATE entity SET ..., version = version + 1 WHERE id = $1 AND version = $2`. If `rowCount === 0`, return `409 Conflict`.
- [ ] **Pessimistic Locking & Queue Claiming**:
  - When claiming limited inventory (E-commerce flash sales, Healthcare appointment slots, Civic event seats) or polling job rows, use `SELECT ... FOR UPDATE SKIP LOCKED` or `FOR NO KEY UPDATE` (to avoid blocking foreign key checks on child tables).
  - Always lock multiple rows in a **deterministic sort order** (e.g., `ORDER BY account_id ASC`) to prevent AB-BA deadlocks across concurrent ledger transfers.
- [ ] **Statement & Lock Timeouts**: Configure `lock_timeout` (e.g., `2s`) and `statement_timeout` (e.g., `10s` for web requests) so a blocked DDL or runaway query fails fast instead of exhausting the connection pool.

---

## 4. Soft Deletes & Partial Unique Indexes

- [ ] **The Soft-Delete Uniqueness Pitfall**: Adding `deleted_at timestamptz NULL` breaks standard `UNIQUE (tenant_id, email)` constraints when a user is soft-deleted and re-invited with the same email.
- [ ] **Partial Unique Index Solution**:
  ```sql
  CREATE UNIQUE INDEX CONCURRENTLY uq_users_tenant_email_active
  ON users (tenant_id, LOWER(email))
  WHERE deleted_at IS NULL;
  ```
- [ ] **Default Query Scoping**: Ensure repository methods and ORM views explicitly filter `WHERE deleted_at IS NULL` unless querying historical archives.

---

## 5. Audit Trails & Multi-Tenancy (PostgreSQL RLS)

- [ ] **Immutable Audit Log**: For regulated operations (FinTech ledger changes, Healthcare EHR access/HIPAA audit, SaaS admin privilege escalation), write append-only audit records (`actor_id`, `actor_ip`, `tenant_id`, `action`, `entity_type`, `entity_id`, `previous_state`, `new_state`, `occurred_at`) within the same database transaction as the mutation.
- [ ] **Row-Level Security (RLS) Defense-in-Depth**:
  - Enable RLS on all multi-tenant tables: `ALTER TABLE records ENABLE ROW LEVEL SECURITY; ALTER TABLE records FORCE ROW LEVEL SECURITY;`.
  - Set the tenant context per transaction: `SET LOCAL app.current_tenant_id = '...';`.
  - Enforce policy:
    ```sql
    CREATE POLICY tenant_isolation_policy ON records
    USING (tenant_id = current_setting('app.current_tenant_id', true)::uuid)
    WITH CHECK (tenant_id = current_setting('app.current_tenant_id', true)::uuid);
    ```

---

## 6. Query Execution Plans, Indexing & Connection Pooling

- [ ] **Index Foreign Keys & Filter Columns**: PostgreSQL automatically indexes `PRIMARY KEY` and `UNIQUE` columns, but **does not automatically index `FOREIGN KEY` columns**. Always create explicit B-Tree indexes on foreign key columns to avoid sequential scans during parent updates/deletes and `JOIN`s.
- [ ] **Covering Indexes (`INCLUDE`) for Hot Read Paths**: On high-throughput lookup queries, append non-filter payload columns via `INCLUDE (status, amount_cents)` to enable **Index-Only Scans** without heap fetches, verified via `EXPLAIN (ANALYZE, BUFFERS)`.
- [ ] **Connection Pooling (`PgBouncer` / `Supavisor` Transaction Mode)**:
  - Size application pools conservatively and route serverless/multi-pod workloads through a transaction-mode connection pooler.
  - Never leak session-level state (`SET app.current_tenant_id` without `LOCAL`) across transaction-pooled connections; always use transaction-scoped `SET LOCAL` inside `BEGIN ... COMMIT`.
