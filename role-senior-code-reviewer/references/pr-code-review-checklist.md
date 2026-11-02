# Staff-Level Pull Request Code Review Checklist

> **Baseline Tooling & Standard Versions (Rule G)**
> - **Language & Type System**: TypeScript `5.7+` (`strict: true`, `noUncheckedIndexedAccess: true`, `exactOptionalPropertyTypes: true`)
> - **Static Analysis & Linting**: ESLint `9.x` (Flat Config) / Biome `1.9+`
> - **Runtime Validation**: Zod `3.24+` / Valibot `1.0+`
> - **Security Baseline**: OWASP Top 10 (`2021`), OWASP API Security Top 10 (`2023`)
> - **Accessibility & Performance**: WCAG `2.2 Level AA`, Core Web Vitals (`LCP < 2.5s`, `INP < 200ms`, `CLS < 0.1`)

---

## 1. Architecture, Modularity & Domain Alignment

- [ ] **Layer & Module Boundary Integrity**:
  - Frontend UI components never import server database drivers, filesystem modules, or raw environment secrets.
  - Shared/domain packages (`src/shared/`, `packages/core/`) remain 100% pure with zero framework or I/O dependencies.
  - Controllers/Route handlers remain thin (parsing, auth, delegating to domain services) rather than embedding 300 lines of mixed SQL and business rules.
- [ ] **Ubiquitous Language & Naming**:
  - Variables, types, API fields, and database columns match the exact domain terminology in `/docs/domain/` (e.g., not mixing `doctor`, `practitioner`, and `provider` for the same entity in a Healthcare app, or `constituency`, `district`, and `ward` in a Civic app).
- [ ] **Backward-Compatible Database Migrations (`Expand-and-Contract`)**:
  - Zero destructive schema changes (dropping/renaming columns or adding `NOT NULL` without a default) in a single deployment step that would break in-flight requests during rolling deploys.
- [ ] **Yagni & Complexity Budget**:
  - No premature abstractions, speculative generic factories, or unused configuration flags. Functions do one clear thing and maintain low cyclomatic complexity.

---

## 2. TypeScript & Strict Type Safety

- [ ] **Zero Type Escapes**:
  - No `any`, `as unknown as T`, `@ts-ignore`, or `@ts-expect-error` without an explicit explanatory comment and runtime guard.
  - Non-null assertions (`user!.profile!.id`) are replaced with explicit narrowing or optional chaining with fallback handling.
- [ ] **Boundary Runtime Validation**:
  - `JSON.parse()`, `fetch()` responses, `localStorage` reads, URL query params, and `process.env` variables are validated at runtime via Zod/Valibot rather than trusted via compile-time `as MyType` casts.
- [ ] **Discriminated Unions & Exhaustiveness**:
  - State machines and result types use discriminated unions (`{ status: 'loading' } | { status: 'success'; data: T } | { status: 'error'; error: E }`) rather than bags of optional booleans (`isLoading?: boolean; isError?: boolean; data?: T`).
  - `switch` statements over unions enforce compile-time exhaustiveness via `satisfies never`.
- [ ] **Branded / Nominal Types for IDs & Units**:
  - Prevent accidental argument swapping (e.g., passing `organizationId` where `userId` is expected, or mixing cents and dollars) using branded types or explicit value objects.

```typescript
// ❌ ANTI-PATTERN: Unsafe cast at network boundary + bag of optional booleans
const data = (await res.json()) as PaymentResponse;
if (data.status === 'paid') { ... }

// ✅ STAFF PATTERN: Runtime schema parse + exhaustive discriminated union
const parsed = PaymentResponseSchema.safeParse(await res.json());
if (!parsed.success) {
  throw new ContractValidationError('Invalid payment payload', parsed.error);
}
```

---

## 3. Concurrency, Race Conditions & Idempotency

- [ ] **Database Read-Modify-Write Race Conditions (TOCTOU)**:
  - Never read a balance/inventory/counter in application memory, add/subtract, and `UPDATE` without an atomic SQL increment (`SET stock = stock - 1 WHERE stock >= 1`), `SELECT ... FOR UPDATE` lock, or optimistic concurrency version check (`WHERE version = expected_version`).
- [ ] **Multi-Step Mutation Transactions**:
  - Any operation writing to multiple tables (e.g., creating an Order, deducting Inventory, and recording an Audit Log) executes inside a single atomic database transaction (`db.transaction(...)`). External network calls (sending emails, charging cards) are placed *outside* the DB transaction lock (or via the Transactional Outbox pattern) to prevent holding DB connections open during slow HTTP calls.
- [ ] **Frontend Stale Closure & Out-of-Order Fetch Races**:
  - Search inputs, filter toggles, and route transitions cancel in-flight requests via `AbortController` (or framework equivalents like TanStack Query / Angular `rxResource` / `switchMap`) so a slow earlier request never overwrites a faster newer response.
- [ ] **Double-Submit & Idempotency Protection**:
  - Mutating buttons disable or ignore duplicate clicks while pending, and critical POST/PUT endpoints enforce `Idempotency-Key` headers or unique constraint guards.

---

## 4. Memory, Resource & Connection Leaks

- [ ] **Event Listener & Subscription Teardown**:
  - Every `window.addEventListener`, `ResizeObserver`, `IntersectionObserver`, `WebSocket`, `setInterval`, or RxJS `Observable.subscribe()` is deterministically cleaned up on component unmount (`AbortSignal`, `useEffect` cleanup, Angular `DestroyRef` / `takeUntilDestroyed()`).
- [ ] **Unbounded In-Memory Caches**:
  - Never store user/request-keyed entries in a module-level `Map` or object without an LRU eviction limit (`maxSize`) and TTL expiration.
- [ ] **Stream & File Handle Closure**:
  - File streams, database cursors, and child processes are closed inside `try / finally` blocks or via `using` (Explicit Resource Management).

---

## 5. Error Handling, Resilience & Observability

- [ ] **No Silent Catch Blocks**:
  - Zero `catch (err) {}` or `catch (err) { console.log(err) }` that swallow failures and leave the UI or database in an inconsistent zombie state.
- [ ] **Timeouts & Circuit Breaking on External I/O**:
  - Every outbound `fetch()`, webhook call, or third-party SDK request sets an explicit `AbortSignal.timeout(ms)` so a hung upstream service cannot exhaust server worker threads.
- [ ] **Structured, Safe Error Responses**:
  - APIs return structured error codes and correlation IDs (`requestId`), never raw stack traces, database table names, or internal file paths.
- [ ] **Contextual Structured Logging**:
  - Errors are logged with structured metadata (`{ requestId, tenantId, operation, durationMs }`) while scrubbing sensitive PII/credentials.

---

## 6. Security & Privacy (OWASP & GDPR Baseline)

- [ ] **Broken Object Level Authorization (BOLA / IDOR)**:
  - Every query/mutation accepting an entity ID (`/api/invoices/:id`, `/api/medical-records/:id`) scopes the database query by the authenticated user's `userId` / `organizationId` (`WHERE id = $1 AND org_id = $2`).
- [ ] **Injection & XSS Prevention**:
  - Zero string concatenation in SQL/NoSQL queries or shell commands (`child_process.exec`).
  - Zero unescaped HTML injection (`dangerouslySetInnerHTML`, `[innerHTML]`, `v-html`) unless sanitized via `DOMPurify` with a strict allowlist.
- [ ] **Mass Assignment Prevention**:
  - Request bodies are explicitly allowlisted via schema picking before passing to ORM `create`/`update` calls (preventing users from injecting `role: "admin"` or `isVerified: true`).
- [ ] **Secrets & PII Hygiene**:
  - Zero hardcoded tokens, private keys, or internal URLs in source or tests.
  - Zero PII (emails, phone numbers, IP addresses, health/financial records) leaked into URL query strings, browser `localStorage`, or unmasked application logs.

---

## 7. Performance & Scalability

- [ ] **Database Query Efficiency (Zero N+1)**:
  - No database queries or `await fetch()` calls inside `for` / `.map()` loops. Use batch loading (`WHERE id = ANY($1)`), joins, or DataLoader patterns.
  - New `WHERE`, `JOIN`, and `ORDER BY` columns on large tables have supporting composite or partial indexes verified via `EXPLAIN ANALYZE`.
- [ ] **Mandatory Pagination**:
  - Every collection endpoint and list query enforces a bounded `limit` (e.g., default `20`, max `100`) using cursor-based or indexed offset pagination.
- [ ] **Frontend Bundle & Rendering Budget**:
  - Heavy libraries (charting, map engines, PDF generators, rich text editors) are dynamically imported (`lazy` / `import()`) rather than bundled into the initial critical path.
  - Images specify `width`, `height`, modern formats (`AVIF`/`WebP`), and `fetchpriority="high"` only on the above-the-fold LCP element.

---

## 8. Accessibility (WCAG 2.2 AA) & UI Craft

- [ ] **All 8 UI States Implemented**:
  - Views handle `Loading (Skeleton)`, `Empty`, `Partial`, `Ideal`, `Overflow / Long Text`, `Error (with Retry)`, `Offline`, and `Permission Denied`.
- [ ] **Keyboard & Screen Reader Navigation**:
  - Interactive elements use native `<button>` and `<a href>` tags (never `<div onClick>`).
  - Modals/drawers trap focus, support `Escape` to close, and restore focus to the trigger element on close.
  - Form inputs have programmatic `<label>` associations and `aria-describedby` links to inline error messages.
- [ ] **Mobile Touch & Motion Craft**:
  - Interactive touch targets are at least `48x48px`; animations use only `transform` and `opacity` and respect `prefers-reduced-motion: reduce`; numeric columns/counters use `tabular-nums`.

---

## 9. Test Quality (Testing Behavior, Not Implementation)

- [ ] **Meaningful Assertions**:
  - Tests assert observable outputs, DOM roles/text, database state, or HTTP responses — never just `expect(mockFn).toHaveBeenCalled()` on internal private helpers.
- [ ] **Negative & Boundary Coverage**:
  - Tests cover not only the happy path, but also invalid inputs (`400`/`422`), unauthorized access (`401`/`403`), not found (`404`), and conflict/duplicate states (`409`).
- [ ] **Deterministic Execution**:
  - Zero `waitForTimeout` sleeps, zero dependence on test execution order, and zero unmocked external network calls.

---

## 10. Living Documentation & Repository Hygiene

- [ ] **Documentation Synchronization (`role-archivist`)**:
  - If the PR alters architectural patterns, API contracts, environment variables, or domain models, the corresponding files in `/docs/adr/`, `/docs/architecture/`, or `/docs/domain/` are updated in the same PR.
- [ ] **Clean Diff & Commit Hygiene**:
  - No leftover debug logs (`console.log("HERE 123")`), commented-out dead code blocks, unrelated lockfile churn, or untracked scratch files.
