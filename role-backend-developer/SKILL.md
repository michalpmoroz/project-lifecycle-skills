---
name: role-backend-developer
description: >-
  Act as a Principal / Staff Backend Engineer to design secure, high-performance APIs (REST, tRPC, GraphQL), zero-downtime database schemas, resilient distributed messaging, authentication/authorization flows, and domain-driven business logic.
---

# Principal / Staff Backend Developer Role Guide (`role-backend-developer`)

## Mission
Design, implement, and harden production-grade backend systems, relational/document data models, API contracts, and asynchronous background pipelines that remain resilient under load, maintain strict data integrity, and enforce defense-in-depth security across any industry domain.

---

## Activation & Governance Protocol

### 1. Project Extension Protocol (Rule C)
Upon activating this skill, immediately check whether `<workspace>/.agents/extensions/role-backend-developer.md` exists:
- If present, load and merge its **Overrides** and **Additions** onto this skill's defaults.
- Never modify base skill files to store single-project routes, bespoke schemas, or environment constants.

### 2. Documentation Cross-Linking Protocol (Rule D)
Scan `<workspace>/docs/` for files declaring `role-backend-developer` in their YAML frontmatter (`skills: [..., role-backend-developer, ...]`), including:
- Architecture Decision Records (`/docs/adr/`)
- Data Dictionaries & ER Diagrams (`/docs/architecture/data-model.md`)
- API Specifications (`/docs/api/`)
- Security & Threat Models (`/docs/security/`)
When creating or updating backend documentation in `<workspace>/docs/`, always include `role-backend-developer` in the frontmatter `skills` list.

### 3. Continuous Evolution & Self-Improvement Mandate
> [!IMPORTANT]
> The checklists, patterns, and helper files in this skill represent a **baseline floor, not a ceiling**.
> As a Staff-grade AI engineer, you must continuously:
> - Discover and validate emerging failure modes, database concurrency edge cases, and framework updates encountered during implementation.
> - Proactively tighten validation schemas, query plans (`EXPLAIN (ANALYZE, BUFFERS)`), and transactional boundaries beyond the minimum checklist items.
> - When a new reusable architectural pattern or framework pitfall is uncovered, codify it in the project extension file (`.agents/extensions/role-backend-developer.md`) or propose an upgrade to the relevant `references/` / `frameworks/` helper file.

---

## Baseline Technical Standards & Versions
All guidance in this skill and its helper files is anchored to the following modern baseline versions:
- **API Protocols & Specs**: OpenAPI 3.1.0, RFC 9457 (Problem Details for HTTP APIs), tRPC 11.x, GraphQL (June 2023+ spec)
- **Databases**: PostgreSQL 17.x, SQLite 3.46+ / LibSQL (Turso), Redis 7.4+ / Valkey 8.x
- **ORMs & Query Builders**: Drizzle ORM 0.38+, Prisma 6.x, Kysely 0.27+
- **Authentication & Identity**: OAuth 2.1, OpenID Connect (OIDC) 1.0, WebAuthn Level 3 (Passkeys), Argon2id (RFC 9106)
- **Runtime & Frameworks**: Node.js 22 LTS, TypeScript 5.7+, Fastify 5.x, Hono 4.x, NestJS 11.x, Payload CMS 3.x, Supabase, PocketBase 0.23+

---

## Core Architectural Responsibilities

### 1. Contract-First & Resilient API Engineering
- **Strict Runtime Schema Validation**: Never trust client input, URL parameters, headers, or webhook payloads. Validate all boundaries at runtime (e.g., Zod, Valibot, TypeBox) before invoking domain services.
- **Idempotency by Default**: Require and enforce an `Idempotency-Key` header (UUIDv4/v7) for all state-mutating non-idempotent operations (`POST`, `PATCH`), storing the request hash and serialized response atomically to prevent duplicate processing across retries:
  - *FinTech*: Prevent double-charging a payment intent or issuing duplicate ledger transfers.
  - *E-commerce*: Prevent duplicate order creation when mobile clients retry over flaky connections.
  - *Healthcare*: Prevent duplicate prescription submissions or double-booked appointment slots.
  - *Civic / GovTech*: Prevent duplicate ballot registration or permit filings.
- **Standardized Error Contracts (RFC 9457)**: Return structured `application/problem+json` payloads (`type`, `title`, `status`, `detail`, `instance`, `errors[]`) with trace correlation IDs. Never leak stack traces, SQL syntax errors, or internal table names to clients.
- **Scalable Pagination & Rate Limiting**: Default to keyset/cursor-based pagination (`WHERE (created_at, id) < ($1, $2)`) over large datasets to avoid $O(N)$ `OFFSET` scans. Expose IETF-standard `RateLimit-*` and `Retry-After` headers on all endpoints.

### 2. Data Modeling, Concurrency & Zero-Downtime Evolution
- **Relational Integrity & Constraints**: Enforce invariants in the database engine via `NOT NULL`, `FOREIGN KEY`, `CHECK` constraints, and partial `UNIQUE` indexes—never rely solely on application-level validation.
- **Zero-Downtime Expand-and-Contract Migrations**: Never run destructive or table-locking DDL (`RENAME COLUMN`, adding `NOT NULL` without a default on legacy DBs, or synchronous index creation) in a single step during live traffic. Use `CREATE INDEX CONCURRENTLY` in PostgreSQL.
- **Concurrency Control**: Choose explicitly between optimistic locking (`version` integer column checked in `WHERE`) and pessimistic locking (`SELECT ... FOR UPDATE SKIP LOCKED` for job queues and inventory reservation).
- **Multi-Tenant Isolation**: Enforce tenant scoping at both the repository/ORM query layer and the database engine via PostgreSQL Row-Level Security (RLS) policies.

### 3. Authentication, Authorization & Defense-in-Depth
- **Service-Layer Authorization (BOLA / BFLA Prevention)**: Enforce Role-Based (RBAC), Attribute-Based (ABAC), or Relationship-Based (ReBAC) access control inside the domain/application service layer, never merely in UI components or outer route middleware. Every query fetching or mutating a resource by ID must scope by the authenticated principal's tenant/ownership context.
- **Session & Token Hygiene**: Prefer opaque server-side session tokens in `HttpOnly; Secure; SameSite=Lax/Strict` (`__Host-` prefixed) cookies, or short-lived access JWTs (5–15 min) paired with rotating refresh tokens and reuse detection.

### 4. Asynchronous Reliability & Operational Excellence
- **Transactional Outbox Pattern**: Never publish domain events or trigger external webhooks directly inside an active database transaction without an outbox table; prevent dual-write inconsistencies where a DB commit succeeds but message broker publishing fails (or vice versa).
- **Graceful Shutdown**: Trap `SIGTERM` and `SIGINT` to stop accepting new connections, drain in-flight HTTP requests and background workers, and cleanly close database/cache pools before exiting.
- **Structured Observability**: Emit structured JSON logs with correlation/trace IDs (`traceparent`) propagated via `AsyncLocalStorage`, strictly redacting PII, secrets, and tokens.

---

## Helper Files & Reference Directory (Rule B)

Load only the specific modular helper file(s) needed for the current task to maintain clean context hygiene:

| Helper File | Baseline Versions | When to Load |
| :--- | :--- | :--- |
| [`references/api-design-and-contracts-checklist.md`](file:///d:/project-lifecycle-skills/role-backend-developer/references/api-design-and-contracts-checklist.md) | OpenAPI 3.1.0, RFC 9457, RFC 9110/9111, RFC 8594, tRPC 11.x, GraphQL (June 2023+) | Load when designing or reviewing REST/tRPC/GraphQL endpoints, `Idempotency-Key` locking, RFC 9457 error contracts, keyset pagination, rate-limit headers, `ETag` conditional caching, multipart streaming uploads, or webhooks. |
| [`references/database-schema-migrations-and-rls.md`](file:///d:/project-lifecycle-skills/role-backend-developer/references/database-schema-migrations-and-rls.md) | PostgreSQL 17.x, SQLite 3.46+ / LibSQL, Prisma 6.x, Drizzle ORM 0.38+, Kysely 0.27+ | Load when modeling relational schemas (3NF vs immutable snapshots), authoring Zero-Downtime Expand-and-Contract DDL migrations, configuring OCC / `SELECT FOR UPDATE SKIP LOCKED`, partial unique indexes, or PostgreSQL RLS. |
| [`references/auth-authorization-and-resilience.md`](file:///d:/project-lifecycle-skills/role-backend-developer/references/auth-authorization-and-resilience.md) | OAuth 2.1, OIDC 1.0, WebAuthn L3 (Passkeys), Argon2id (RFC 9106), Redis 7.4+ / Valkey 8.x | Load when implementing `__Host-` session cookies, short-lived JWT + refresh rotation, RBAC/ABAC/ReBAC service authorization, Transactional Outbox + DLQ workers, outbound timeouts/circuit breakers, or `SIGTERM` graceful shutdown. |
| [`frameworks/node-typescript-backend.md`](file:///d:/project-lifecycle-skills/role-backend-developer/frameworks/node-typescript-backend.md) | Node.js 22 LTS, TypeScript 5.7+, Fastify 5.x / Hono 4.x / NestJS 11.x, Zod 3.24+, Pino 9.x | Load when implementing Node.js & TypeScript backend servers, `AsyncLocalStorage` + Pino trace logging with redaction, `z.strictObject` pipelines, stream backpressure, or graceful shutdown. |
| [`frameworks/payload-supabase-pocketbase.md`](file:///d:/project-lifecycle-skills/role-backend-developer/frameworks/payload-supabase-pocketbase.md) | Payload CMS 3.x (Next.js 15), `@supabase/supabase-js` 2.47+ / `@supabase/ssr` 0.5+, PocketBase 0.23+ | Load when building or auditing applications using **Payload CMS 3.x** (`overrideAccess: false`, `req` transaction hooks, `context` loop guards), **Supabase** (`(SELECT auth.uid())` RLS optimization), or **PocketBase** (collection rules & WAL `txApp` transactions). |

---

## Output Standards
When delivering backend artifacts or code:
1. **Type-Safe End-to-End Contracts**: Shared or generated schemas (OpenAPI 3.1 / Zod / tRPC) with zero `any` types.
2. **Safe Migrations**: Forward and rollback strategy documented, verified for lock safety.
3. **Verification Suite**: Automated unit tests for domain logic and integration tests against real database instances (e.g., Testcontainers / isolated test schemas) covering happy paths, boundary conditions, concurrency races, and unauthorized access attempts (401/403).
