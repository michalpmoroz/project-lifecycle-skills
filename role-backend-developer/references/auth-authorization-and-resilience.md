# Authentication, Authorization, Distributed Messaging & Resilience Checklist (`references/auth-authorization-and-resilience.md`)

> **Baseline Versions**: OAuth `2.1`, OpenID Connect (OIDC) `1.0`, WebAuthn Level `3` (Passkeys), Argon2id (RFC `9106`), Redis `7.4+` / Valkey `8.x`, OpenTelemetry `1.x`.
> **Mandate**: Load and enforce this checklist when designing authentication flows, service-layer access control (RBAC/ABAC/ReBAC), transactional outbox queues, downstream resilience, or graceful shutdown handlers.

> [!NOTE]
> **Continuous Evolution Mandate**: This checklist is a baseline floor, not a static ceiling. Engineers must proactively evaluate privilege escalation vectors, token revocation races, message redelivery side effects, and cascading failure modes.

---

## 1. Session Tokens vs. Short-Lived JWT + Refresh Rotation

- [ ] **Browser Session Cookies**: Use opaque high-entropy (256-bit CSPRNG) session IDs stored hashed (SHA-256) in the DB/Redis, delivered via `__Host-session` cookie (`Secure; HttpOnly; SameSite=Lax; Path=/`).
- [ ] **Token-Based Auth (Mobile / Third-Party APIs)**:
  - Issue short-lived Access Tokens (5–15 minutes TTL) signed with asymmetric keys (`EdDSA` / `ES256`, never weak symmetric secrets or `alg: none`).
  - Issue one-time-use Refresh Tokens with **Refresh Token Rotation** and **Family Reuse Detection**: if a previously rotated refresh token is presented again, immediately revoke the entire token family for that user session.
- [ ] **Password & Passkey Standards**:
  - Support **WebAuthn / Passkeys** for phishing-resistant authentication.
  - Hash passwords exclusively with **Argon2id** (minimum OWASP parameters: `m=19456` (19 MiB), `t=2`, `p=1` or `m=47104` (46 MiB), `t=1`, `p=1`).

---

## 2. Service-Layer Authorization (RBAC / ABAC / ReBAC)

- [ ] **Mandatory Policy Evaluation**: Every domain service method must accept an explicit `ActorContext` (`userId`, `tenantId`, `roles`, `permissions`, `scopes`) and verify authorization before reading or mutating records.
- [ ] **Prevent BOLA / IDOR**: Never query `WHERE id = $1` alone. Always query `WHERE id = $1 AND tenant_id = $2` (and verify ownership/relationship).
- [ ] **Prevent Mass Assignment**: Never pass raw request bodies (`req.body`) directly into ORM `.create()` or `.update()` calls. Explicitly construct DTOs permitting only user-editable fields (excluding `role`, `tenantId`, `isVerified`, `balance`, etc.).

---

## 3. Transactional Outbox Pattern & Dead-Letter Queues (DLQ)

- [ ] **Outbox Table**: Write domain events into an `outbox_events` table (`id`, `aggregate_type`, `aggregate_id`, `event_type`, `payload`, `trace_parent`, `created_at`, `processed_at`) inside the same local ACID transaction that mutates business state.
- [ ] **Relay Worker & Idempotent Consumers**:
  - A background relay (using `SELECT ... FOR UPDATE SKIP LOCKED` or PostgreSQL logical replication WAL tailing) publishes outbox events to the message broker and marks `processed_at = NOW()`.
  - Because outbox delivery guarantees **at-least-once** semantics, every downstream event consumer must be **idempotent** (tracking processed `event_id` in an `inbox_processed_events` table).
- [ ] **Dead-Letter Queues (DLQ) & Poison Pill Handling**:
  - Configure a maximum delivery retry count (e.g., 5 attempts with exponential backoff + jitter).
  - Route permanently failing messages to a DLQ with full error diagnostics and trigger an operational alert. Never block an entire partition on a single poison message.

---

## 4. Downstream Timeouts, Circuit Breakers & Retry Budgets

- [ ] **Mandatory Outbound Timeouts (`AbortSignal.timeout`)**: Never issue an outbound `fetch()`, gRPC call, or third-party SDK request (e.g., payment gateway, EHR FHIR API, SMS provider) without an explicit deadline (`signal: AbortSignal.timeout(3000)`). Unbounded sockets exhaust worker threads and connection pools during upstream brownouts.
- [ ] **Exponential Backoff with Full Jitter**: Retry only transient network errors or `429` / `502` / `503` / `504` responses on idempotent operations using capped exponential backoff with random jitter (`sleep = random(0, min(capMs, baseMs * 2 ** attempt))`) so recovering services are not hammered by synchronized retry storms.
- [ ] **Circuit Breaking**: Wrap flaky external dependencies in a Circuit Breaker (`CLOSED` -> `OPEN` -> `HALF_OPEN`) that fails fast when error rates exceed threshold, returning graceful degraded fallbacks or queued processing.

---

## 5. Graceful Shutdown & Connection Pool Hygiene

- [ ] **Signal Trapping (`SIGTERM` / `SIGINT`)**:
  1. Immediately fail readiness probes (`/health/ready` returns `503`) so load balancers stop routing new traffic.
  2. Stop accepting new HTTP connections (`server.close()`) and pause background queue consumers.
  3. Wait for in-flight requests and active jobs to finish within a bounded grace period (e.g., `20s`, shorter than Kubernetes/container `terminationGracePeriodSeconds: 30`).
  4. Flush telemetry buffers (OpenTelemetry spans/logs) and close database/Redis connection pools (`pool.end()`).
  5. Exit cleanly with status code `0`.
