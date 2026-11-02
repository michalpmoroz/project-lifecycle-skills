# SQL, Backend Service & Automated Test Suite Style Rules

> **Baseline Specification & Tooling Versions (Rule G)**
> - **Database & SQL Standards**: ISO/IEC 9075 `SQL:2023`, PostgreSQL `17`, SQLite `3.47+`, SQLFluff `3.2+`
> - **Backend Runtimes & Validation**: Node.js `22 LTS` / Python `3.13+` / Go `1.23+`, Zod `3.24+`, OpenAPI `3.1` / RFC `9457`
> - **Test Runners & Linters**: Vitest `2.x` (`eslint-plugin-vitest`), Jest `29+`, Playwright `1.49+` (`eslint-plugin-playwright`), Testing Library `16+`

---

## 1. SQL Style & Database Schema Naming Rules

Every database migration, schema definition (Drizzle, Prisma, Kysely, SQLAlchemy, sqlc), and raw SQL query must enforce these naming and formatting invariants:

### 1.1 Schema Identifier & Casing Matrix

| Database Object | Casing & Plurality Rule | Multi-Domain Examples (FinTech, Healthcare, Civic, E-commerce, SaaS) | Anti-Patterns to Ban |
| :--- | :--- | :--- | :--- |
| **SQL Reserved Keywords & Functions** | **`UPPERCASE`** | `SELECT`, `FROM`, `INNER JOIN`, `WHERE`, `GROUP BY`, `RETURNING`, `COALESCE()`, `COUNT()` | Lowercase keywords mixed randomly (`select id From users where...`) |
| **Schemas, Tables, Views & Columns** | **`lower_snake_case`** (unquoted ASCII) | `ledger_entries`, `patient_encounters`, `polling_stations`, `tenant_subscriptions` | Quoted `"CamelCase"` or `"PascalCase"` identifiers (which force permanent double-quoting in every PostgreSQL query!) |
| **Table & View Names** | **Plural `lower_snake_case` nouns** | `invoices`, `clinical_observations`, `ballot_contests`, `fulfillment_shipments` | Singular table names (`user`, `order` — which collides with the `ORDER BY` reserved keyword!) |
| **Join / Many-to-Many Tables** | `<plural_table_a>_<plural_table_b>` or domain noun | `practitioners_specialties`, `role_permissions`, `order_line_items` | `tbl_user_role_map`, `link1` |
| **Primary Key Columns** | Singular **`id`** (`uuid` or `bigint`) | `id UUID PRIMARY KEY DEFAULT gen_random_uuid()` | Naming the PK `orders_id` on the `orders` table |
| **Foreign Key Columns** | **`<singular_target_table>_id`** | `organization_id`, `patient_id`, `constituency_id`, `parent_invoice_id` | `fk_org`, `orgId`, `id_organization` |
| **Timestamp Columns** | **`<past_participle_verb>_at`** (`TIMESTAMPTZ`) | `created_at`, `updated_at`, `deleted_at`, `settled_at`, `verified_at` | `create_date`, `time`, `timestamp`, or using timezone-naive `TIMESTAMP WITHOUT TIME ZONE` |
| **Calendar Date Columns** | **`<noun>_date`** (`DATE`) | `birth_date`, `election_date`, `statement_period_start_date` | Storing calendar dates as strings (`VARCHAR`) |
| **Boolean Columns** | **`is_<adjective>` / `has_<noun>`** (`NOT NULL`) | `is_verified`, `is_archived`, `has_two_factor_enabled` | Nullable booleans (`NULL` three-valued logic traps) or negative names (`is_not_active`) |
| **Monetary & Quantity Columns** | **`<noun>_minor_units` / `<noun>_cents` / `<noun>_bps`** (`BIGINT`) | `subtotal_minor_units`, `tax_amount_cents`, `interest_rate_bps` | Using `FLOAT` or `REAL` for currency (causes IEEE 754 rounding corruption) |

### 1.2 Deterministic Constraint & Index Naming Prefixes
Never allow the database engine to auto-generate opaque constraint names (`users_col1_col2_key1`). Name every constraint explicitly so error logs and migrations remain self-documenting:
- **Primary Key**: `pk_<table>` (e.g., `pk_ledger_entries`)
- **Foreign Key**: `fk_<table>_<column>_<referenced_table>` (e.g., `fk_invoices_organization_id_organizations`)
- **Unique Constraint**: `uq_<table>_<column_list>` (e.g., `uq_tenant_memberships_tenant_id_user_id`)
- **Check Constraint**: `chk_<table>_<rule_summary>` (e.g., `chk_invoices_total_minor_units_non_negative`)
- **Index**: `idx_<table>_<column_list>` (e.g., `idx_patient_encounters_patient_id_scheduled_at`)

### 1.3 SQL Query Formatting & Readability Rules
1. **Strictly Ban `SELECT *` in Application Queries**:
   - Explicitly list every required column (`SELECT id, organization_id, status, total_minor_units FROM invoices`). `SELECT *` breaks positional deserializers when columns are added, wastes network/memory bandwidth on large `TEXT`/`JSONB` columns, and defeats index-only scans.
2. **Prefer CTEs (`WITH ... AS`) Over Deeply Nested Subqueries**:
   - Factor multi-step aggregations or filtering into named Common Table Expressions (`WITH active_subscriptions AS (...)`) rather than 4-level nested `FROM (SELECT ... FROM (SELECT ...))` pyramids.
3. **Explicit `INNER JOIN` / `LEFT JOIN` & `AS` Aliases**:
   - Always write `INNER JOIN` or `LEFT JOIN` explicitly (never bare `JOIN` or legacy comma-separated `FROM a, b WHERE a.id = b.a_id`).
   - Always include the `AS` keyword when aliasing columns or tables (`SUM(e.amount_minor_units) AS total_settled_minor_units`).
4. **One Column / Predicate Per Line in Multi-Condition Queries**:
   - Align top-level clauses (`SELECT`, `FROM`, `INNER JOIN`, `WHERE`, `GROUP BY`, `ORDER BY`, `LIMIT`) at the left margin and indent their items by `2 spaces`.

```sql
-- ❌ ANTI-PATTERN: Lowercase keywords, SELECT *, implicit join, nested subquery, ambiguous column names
select * from orders o, customers c
where o.customerId = c.id and o.status = 'paid'
and o.total > (select avg(total) from orders where created > '2026-01-01');

-- ✅ STAFF PATTERN: Explicit CTE, UPPERCASE keywords, lower_snake_case, explicit columns & units
WITH regional_order_averages AS (
  SELECT
    shipping_region_code,
    AVG(total_minor_units) AS avg_region_total_minor_units
  FROM customer_orders
  WHERE created_at >= '2026-01-01T00:00:00Z'
  GROUP BY shipping_region_code
)
SELECT
  co.id,
  co.organization_id,
  co.customer_id,
  co.total_minor_units,
  co.settled_at
FROM customer_orders AS co
INNER JOIN regional_order_averages AS roa
  ON co.shipping_region_code = roa.shipping_region_code
WHERE
  co.organization_id = $1
  AND co.order_status = 'settled'
  AND co.total_minor_units > roa.avg_region_total_minor_units
ORDER BY co.settled_at DESC
LIMIT 50;
```

---

## 2. Backend Service, Error & Logging Style Rules

1. **Thin Transport Controllers, Rich Domain Services**:
   - Route handlers / controllers (`*.controller.ts`, `*.router.ts`) must not exceed `35–40` lines and perform exactly 4 steps:
     1. Parse and validate inbound parameters/body via schema (`Schema.parse(req.body)`).
     2. Verify authentication & tenant/resource authorization context.
     3. Delegate execution to a pure domain/application service (`await invoiceService.issueInvoice(command)`).
     4. Return a typed HTTP status and response DTO.
2. **Structured Domain Error Hierarchy**:
   - Never throw generic `new Error('Not found')` and regex-match `err.message.includes('Not found')` in middleware to guess the HTTP status code!
   - Define a base `DomainError` / `AppError` class carrying explicit `readonly code: ErrorCode`, `readonly httpStatus: number`, `readonly metadata?: Record<string, unknown>`, and standard `{ cause }`.
3. **Structured Logging Field Conventions**:
   - Always pass a structured metadata object as the **first** argument and a static human-readable message string as the **second** argument (`logger.info({ requestId, tenantId, invoiceId, durationMs }, 'Invoice settled successfully')`).
   - Never interpolate dynamic IDs into the log message string (`logger.info(\`Invoice \${id} settled\`)`), which fragments log aggregation cardinality in Datadog/Loki/OpenTelemetry.

---

## 3. Automated Test Suite Style Rules (Unit, Integration & Playwright E2E)

Test code is first-class production code. Unreadable or non-deterministic tests erode confidence and slow down delivery.

### 3.1 Test Naming & Behavioral Descriptions
- **File Naming**: Co-locate unit tests as `<module-name>.spec.ts` (or `.test.ts`) and place browser E2E suites in `e2e/<user-journey>.e2e.ts`.
- **Behavioral `it()` / `test()` Titles**:
  - Write test descriptions in the present tense starting with an active verb that states **the business outcome and the condition**:
    - Good: `it('rejects wire transfer when Idempotency-Key belongs to a different payload hash', ...)`
    - Good: `it('calculates pediatric amoxicillin dosage capped at the adult maximum ceiling', ...)`
    - Bad: `it('works')`, `it('test transferError')`, `it('calls repo.findById once')` (testing mocks instead of behavior).

### 3.2 Mandatory Arrange – Act – Assert (AAA) Block Separation
Every test body must be visually separated into three distinct paragraphs using single blank lines:
1. **Arrange**: Construct isolated test inputs using factory builders (`buildOrder({ ... })`) and configure boundary stubs.
2. **Act**: Invoke the single target function, API endpoint, or user interaction under test.
3. **Assert**: Verify the returned output, state transition, or user-visible DOM outcome.

### 3.3 Strict Test Suite Lint Guardrails (`eslint-plugin-vitest` / `eslint-plugin-playwright`)

| # | Test Style Rule | Defect Prevented |
| :--- | :--- | :--- |
| **TS-T01** | **Ban Conditional Logic Inside Tests (`no-conditional-in-test`)** | Never put `if / else`, `switch`, `for`, or `try / catch` inside a test body. A test with an `if` branch silently passes when the condition is false! Split different conditions into separate `it()` blocks or use `it.each(...)`. |
| **TS-T02** | **Ban `.only` and `.skip` Without Ticket Reference (`no-focused-tests`)** | Committing `it.only` / `test.only` silently disables the rest of the CI test suite! Block `*.only` at pre-commit and CI lint gates. |
| **TS-T03** | **Require Explicit Assertions (`expect-expect`)** | Every test must execute at least one `expect(...)` assertion (preventing false-positive tests that merely call a function without checking the result). |
| **TS-T04** | **Prefer Factory Builders Over Shared Mutable `beforeEach` State** | Shared `let mockUser: User;` mutated across 15 nested `beforeEach` blocks causes order-dependent test pollution. Use pure factory helpers (`const patient = buildPatient({ isInsured: false });`) directly inside the **Arrange** step. |
| **TS-T05** | **Enforce Deterministic Clocks & UUIDs in Unit Tests** | Never rely on real wall-clock `new Date()` timing or `setTimeout(..., 500)` in unit tests. Inject a clock interface or use `vi.useFakeTimers()` + `vi.setSystemTime(new Date('2026-11-02T12:00:00Z'))`. |

```typescript
// ❌ ANTI-PATTERN: Shared mutable state + `try/catch` conditional in test + vague test title
let order: any;
beforeEach(() => {
  order = { id: '1', total: 100, status: 'pending' };
});

it('test refund', async () => {
  order.status = 'settled';
  try {
    const res = await refundOrder(order, 150);
    if (res) {
      expect(res.error).toBeDefined();
    }
  } catch (e) {
    expect(e).toBeTruthy();
  }
});

// ✅ STAFF PATTERN: Pure factory builder + strict AAA separation + explicit error assertion
it('rejects partial refund when requested amount exceeds settled order total', async () => {
  // Arrange
  const settledOrder = buildCustomerOrder({
    id: asOrderId('ord_01J9X2K8M'),
    status: 'settled',
    totalMinorUnits: 10_000,
  });

  // Act & Assert
  await expect(
    issueOrderRefund({
      order: settledOrder,
      refundAmountMinorUnits: 15_000,
      reasonCode: 'CUSTOMER_REQUEST',
    }),
  ).rejects.toThrowError(
    expect.objectContaining({
      code: 'REFUND_EXCEEDS_SETTLED_TOTAL',
      httpStatus: 422,
    }),
  );
});
```
