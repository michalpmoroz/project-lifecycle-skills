# Backend API Design, Protocol Engineering & Contracts Checklist (`references/api-design-and-contracts-checklist.md`)

> **Baseline Versions**: OpenAPI `3.1.0`, RFC `9457` (Problem Details for HTTP APIs), RFC `9110` / RFC `9111` (HTTP Semantics & Caching), RFC `8594` (`Sunset` Header), tRPC `11.x`, GraphQL (June `2023+` spec).
> **Mandate**: Load and enforce this checklist when designing, implementing, or reviewing REST, tRPC, GraphQL, or Webhook APIs.

> [!NOTE]
> **Continuous Evolution Mandate**: This checklist is a baseline floor, not a static ceiling. Engineers must proactively identify domain-specific failure modes, payload edge cases, and protocol abuse vectors beyond the items listed below.

---

## 1. Idempotency & State Mutation Safety

- [ ] **`Idempotency-Key` Enforcement**: Require a client-generated `Idempotency-Key` header (UUIDv4 or UUIDv7, max 64 chars) on all non-idempotent state-mutating endpoints (`POST`, `PATCH`).
- [ ] **Atomic Fingerprint & State Locking**:
  - Store `(tenant_id, idempotency_key, request_method, request_path, payload_sha256, status, response_code, response_body, locked_until, created_at)` in a dedicated table or Redis/Valkey key with a 24–72 hour TTL.
  - If a concurrent duplicate request arrives while the first is still `IN_PROGRESS`, return `409 Conflict` (or `425 Too Early`) with a `Retry-After` header.
  - If a duplicate key arrives with a **different** `payload_sha256`, reject immediately with `422 Unprocessable Content` (`idempotency_key_payload_mismatch`).
- [ ] **Multi-Domain Application**:
  - *FinTech*: Payment capture, wire transfer initiation, ledger journal posting.
  - *E-commerce*: Checkout submission, coupon redemption, inventory allocation.
  - *Healthcare*: Lab order creation, electronic prescription dispatch.
  - *Civic / GovTech*: Permit application submission, public comment filing.

---

## 2. Standardized Error Responses (RFC 9457)

- [ ] **Problem Details Media Type**: Return `Content-Type: application/problem+json` for all `4xx` and `5xx` responses adhering to **RFC 9457**.
- [ ] **Canonical Error Schema**:
  ```json
  {
    "type": "https://api.example.com/errors/validation-failed",
    "title": "Request Validation Failed",
    "status": 422,
    "detail": "One or more fields failed validation constraints.",
    "instance": "/v1/orders/ord_01J9X2K8M",
    "traceId": "00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01",
    "errors": [
      {
        "pointer": "#/items/0/quantity",
        "code": "value_below_minimum",
        "message": "Quantity must be at least 1."
      }
    ]
  }
  ```
- [ ] **Zero Internal Leakage**: Never expose database error codes (e.g., `23505` raw constraint names), stack traces, file paths, or upstream internal hostnames in production responses. Map known DB constraint errors cleanly to domain errors (`409 Conflict`).

---

## 3. Cursor / Keyset Pagination & Filtering

- [ ] **Prohibit Unbounded Queries**: Every collection endpoint must enforce a hard maximum page size (e.g., `limit <= 100`, default `25`).
- [ ] **Keyset (Seek) Pagination over `OFFSET`**:
  - Avoid `OFFSET N` for deep pagination ($N > 1000$ forces the DB to scan and discard $N$ rows).
  - Use deterministic composite cursors encoding `(sort_column_value, unique_id)` (e.g., base64url-encoded `{ "createdAt": "2026-03-15T10:00:00Z", "id": "01J9X..." }`).
  - Query pattern:
    ```sql
    SELECT id, status, created_at
    FROM records
    WHERE tenant_id = $1
      AND (created_at, id) < ($2::timestamptz, $3::uuid)
    ORDER BY created_at DESC, id DESC
    LIMIT $4;
    ```
  - Back the query with a matching composite index: `CREATE INDEX CONCURRENTLY idx_records_tenant_cursor ON records (tenant_id, created_at DESC, id DESC);`.

---

## 4. Rate Limiting & Backpressure Headers

- [ ] **Tiered Rate Limits**: Enforce rate limits per IP (unauthenticated), per User/API Key (authenticated), and per Tenant (organization quota) using Token Bucket or Sliding Window Counter algorithms in Redis/Valkey.
- [ ] **Standard Headers**: Include `RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset` (IETF draft standard) on responses, and `Retry-After` (in seconds) on `429 Too Many Requests`.
- [ ] **Cost-Based Rate Limiting (GraphQL / Search)**: Bound GraphQL query depth (e.g., max depth `7`), alias counts, and computed query complexity before execution to prevent single-request DoS.

---

## 5. Content Negotiation, Conditional Caching (`ETag`) & Streaming Uploads

- [ ] **Strict Content Negotiation**:
  - Verify `Content-Type` on all requests carrying a body (`POST`, `PUT`, `PATCH`); reject unexpected media types with `415 Unsupported Media Type`.
  - Respect the `Accept` header (`application/json`, `application/problem+json`, `text/csv`) and return `406 Not Acceptable` when the client requests an unsupported representation.
- [ ] **Conditional Requests (`ETag` / `If-None-Match` & `If-Match`)**:
  - Emit an `ETag` header (derived from `record.version` or `updated_at` + ID hash, e.g., `ETag: "v3-01J9X2K8M"`) on `GET` resource endpoints.
  - When a client sends `If-None-Match: "v3-01J9X2K8M"` and the resource is unchanged, short-circuit serialization and return `304 Not Modified` with an empty body.
  - Support `If-Match: "v3-01J9X2K8M"` on `PUT` / `PATCH` / `DELETE` endpoints to enforce optimistic concurrency over HTTP—returning `412 Precondition Failed` if another actor modified the resource in the interim.
- [ ] **Multipart Streaming & Direct-to-Object-Storage Uploads**:
  - Never buffer large multipart file uploads (`> 5–10 MB`) in application process memory.
  - For large media or document uploads (Healthcare DICOM/PDFs, E-commerce product videos, Civic public record archives), issue short-lived **Presigned S3/R2 Upload URLs** with strict `Content-Length` and `Content-Type` policy conditions, or stream bytes directly through a backpressure-aware pipeline (`stream.pipeline`) with hard byte-size and part-count limits.

---

## 6. Webhook Delivery & Signature Verification

- [ ] **Inbound Webhook Verification**:
  - Read the **raw unparsed request body bytes** before JSON middleware parsing.
  - Verify the HMAC-SHA256 (or Ed25519) signature over `${timestamp}.${rawBody}`.
  - Use constant-time comparison (`crypto.timingSafeEqual`) to prevent timing attacks.
  - **Replay Protection**: Reject webhook deliveries where the signed `timestamp` header deviates from server UTC time by more than `300 seconds` (5 minutes), and record processed `webhook_event_id` values to ensure idempotent handling.
- [ ] **Outbound Webhook Resilience**: Dispatch outbound webhooks asynchronously via a queue with exponential backoff + jitter, circuit breaking per destination endpoint, and a customer-visible delivery log.

---

## 7. API Versioning & Evolution

- [ ] **Additive Evolution**: Evolve schemas backward-compatibly by only adding optional request fields or new response attributes.
- [ ] **Deprecation Lifecycle**: When breaking changes are unavoidable, introduce a new version (`/v2/...` or header/schema version) and emit `Deprecation: @<unix-timestamp>` and `Sunset: <HTTP-date>` headers (RFC 8594) alongside a `Link: <...>; rel="successor-version"` header.
