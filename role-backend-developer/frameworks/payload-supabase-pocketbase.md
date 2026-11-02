# Payload CMS 3.x, Supabase & PocketBase Implementation Guide (`frameworks/payload-supabase-pocketbase.md`)

> **Baseline Versions**: Payload CMS `3.x` (Next.js 15 App Router native, Drizzle/Postgres & MongoDB adapters), Supabase (`@supabase/supabase-js` `2.47+`, `@supabase/ssr` `0.5+`, PostgreSQL `17`), PocketBase `0.23+` (Go / Embedded SQLite WAL).
> **Mandate**: Load this helper whenever architecting, implementing, or auditing backends built on Payload CMS 3.x, Supabase, or PocketBase.

> [!NOTE]
> **Continuous Evolution Mandate**: Headless CMS and BaaS platforms provide rapid velocity but harbor subtle default access-control pitfalls. Always verify row-level, field-level, and transaction-level isolation with integration tests running as non-admin principals.

---

## 1. Payload CMS 3.x Security & Architecture Best Practices

Payload CMS 3.x runs natively inside the Next.js App Router and exposes a powerful Local API (`payload.find`, `payload.create`, `payload.update`, `payload.delete`). **Misconfiguring the Local API or Hooks is the #1 source of critical security and data-corruption bugs in Payload applications.**

### 1.1 Rule #1: Local API Access Control (`overrideAccess: false`)
> [!CAUTION]
> By default, Payload's Local API (`payload.find()`, `payload.update()`, etc.) sets `overrideAccess: true`, **completely bypassing all collection and field access control rules** because it assumes a trusted server context.
> Whenever calling the Local API on behalf of an authenticated user (or an unauthenticated public visitor), you **MUST** explicitly pass `user` AND set `overrideAccess: false`.

```typescript
// ❌ CRITICAL VULNERABILITY: Ignores collection access control! Any user can read restricted records!
const records = await payload.find({
  collection: 'patient-records', // or 'orders', 'ballots', 'financial-statements'
  user: currentUser,
});

// ✅ SECURE: Enforces collection & field-level access control rules for currentUser
const records = await payload.find({
  collection: 'patient-records',
  user: currentUser,
  overrideAccess: false,
  where: {
    tenant: { equals: currentUser.tenantId },
  },
});
```

### 1.2 Rule #2: Transaction Propagation in Hooks (`req` Passing)
> [!WARNING]
> In Payload CMS 3.x (backed by Drizzle/Postgres or MongoDB transactions), an active database transaction is attached to `req.transactionID`.
> If a `beforeChange`, `afterChange`, or `afterDelete` hook performs nested `req.payload.create/update/find` calls **without passing `req`**, those nested operations execute **outside** the parent transaction—causing partial writes on rollback or deadlocks!

```typescript
import type { CollectionAfterChangeHook } from 'payload';

export const writeAuditLogOnUpdate: CollectionAfterChangeHook = async ({
  doc,
  previousDoc,
  operation,
  req,
}) => {
  // ✅ ALWAYS pass `req` to nested Local API calls inside hooks so they share the active DB transaction!
  await req.payload.create({
    collection: 'audit-logs',
    req, // <-- CRITICAL: binds to the same atomic transaction
    data: {
      action: operation,
      entityId: String(doc.id),
      previousState: previousDoc,
      newState: doc,
      actor: req.user?.id ?? null,
    },
  });

  return doc;
};
```

### 1.3 Rule #3: Preventing Infinite Hook Loops (`context` Guard)
When an `afterChange` hook updates the same collection (or a mutually triggering collection), guard against infinite recursion using `req.context`:

```typescript
export const syncDerivedStatus: CollectionAfterChangeHook = async ({ doc, req, context }) => {
  if (context.skipDerivedStatusSync) return doc;

  await req.payload.update({
    collection: 'orders',
    id: doc.id,
    req,
    context: {
      ...context,
      skipDerivedStatusSync: true, // Prevents recursive loop
    },
    data: {
      computedSummary: `${doc.referenceCode}-${doc.status}`,
    },
  });

  return doc;
};
```

### 1.4 Rule #4: Query-Level Access Control vs. Boolean Access Control
- In `access.read` / `access.update` / `access.delete`, return a **Query constraint (`Where` object)** rather than a simple boolean whenever row-level filtering is needed:
```typescript
import type { Access } from 'payload';

export const tenantOrAdminAccess: Access = ({ req: { user } }) => {
  if (!user) return false;
  if (user.roles?.includes('super-admin')) return true;

  // Row-level security filter enforced automatically on find, findByID, update, and delete
  return {
    tenant: {
      equals: user.tenantId,
    },
  };
};
```
- **Field-Level Access Control**: Note that Field-level access functions (`fields[].access.read/update`) only support returning a `boolean` (not a `Where` query). Always protect sensitive attributes (`roles`, `internalNotes`, `stripeCustomerId`, `ssnHash`) with explicit field-level `update: ({ req: { user } }) => Boolean(user?.roles?.includes('admin'))` so users cannot escalate privileges via self-update endpoints.

---

## 2. Supabase (PostgreSQL + RLS + Edge Functions) Best Practices

### 2.1 Service Role Key vs. Anon Key Isolation
- **Never expose `SUPABASE_SERVICE_ROLE_KEY`** to browser bundles, mobile apps, or client-side code. The `service_role` key bypasses all PostgreSQL Row-Level Security (RLS).
- In SSR / Edge / API routes acting on behalf of a user, instantiate a request-scoped Supabase client using `@supabase/ssr` with the user's JWT so Postgres RLS policies evaluate `auth.uid()` and `auth.jwt()` accurately.

### 2.2 High-Performance RLS Policy Authoring
- **Wrap `auth.uid()` and `auth.jwt()` in `(SELECT ...)`**: In PostgreSQL, calling `auth.uid()` directly in an RLS policy can be evaluated per-row. Wrapping it as `(SELECT auth.uid())` forces Postgres to evaluate it once as an `initPlan` constant, yielding up to 10x–100x faster queries on large tables:
```sql
-- ❌ SLOW: Evaluates auth.uid() on every scanned row
CREATE POLICY "Users can view own records" ON records
FOR SELECT TO authenticated
USING (user_id = auth.uid());

-- ✅ FAST: Cached per-query via initPlan + indexed column
CREATE POLICY "Users can view own records" ON records
FOR SELECT TO authenticated
USING (user_id = (SELECT auth.uid()));
```
- **Index Every RLS Column**: Every column referenced inside a `USING` or `WITH CHECK` clause (`user_id`, `tenant_id`, `org_id`) **must** have a B-Tree index.
- **Security Definer Helper Functions**: When checking membership in a join table (`org_members`), avoid recursive RLS evaluation by wrapping the lookup in a `SECURITY DEFINER` function with `SET search_path = public, pg_temp` and `STABLE` volatility.

---

## 3. PocketBase (Go / Embedded SQLite) Best Practices

### 3.1 Collection API Rules as Database Filters
- PocketBase Collection API Rules (`listRule`, `viewRule`, `createRule`, `updateRule`, `deleteRule`) act as both access gates and SQL `WHERE` filters.
- **Lock Admin-Only Collections**: Leave a rule as `null` (Admin Only) rather than `""` (empty string = Public Access to anyone!) whenever an endpoint should not be publicly accessible.
- **Prevent Privilege Escalation on `users` / Custom Collections**: In `createRule` and `updateRule`, explicitly prohibit users from modifying protected fields using `:isset` modifier checks:
  ```text
  @request.auth.id != "" && id = @request.auth.id && (@request.body.role:isset = false) && (@request.body.tenantId:isset = false)
  ```

### 3.2 Transactional Safety & Concurrency in PocketBase Hooks
- Ensure SQLite is configured in **WAL (Write-Ahead Logging)** mode (`PRAGMA journal_mode = WAL; PRAGMA busy_timeout = 5000;`) to allow concurrent reads during writes.
- When extending PocketBase via Go plugins or JS `pb_hooks`, wrap multi-record mutations in `app.RunInTransaction(func(txApp core.App) error { ... })` (or `$app.runInTransaction((txApp) => { ... })` in JSVM) and **always use `txApp` (never the outer `app`)** inside the callback to avoid SQLite database-locked deadlocks.
