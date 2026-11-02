# OWASP Web, API, Access Control & Real-Time Security Audit Checklist (`references/owasp-web-api-and-access-checklist.md`)

> **Baseline Standards**: OWASP Top 10:2021, OWASP API Security Top 10:2023, OWASP ASVS v4.0.3 / v5.0 (Level 2/3), NIST SP 800-63B, RFC 6455 (WebSockets).
> **Mandate**: Load and enforce this checklist when auditing or implementing access control, input handling, outbound HTTP fetches (SSRF), authentication/sessions, HTTP headers, file uploads, or WebSockets.

> [!IMPORTANT]
> **Continuous Evolution Mandate**: This checklist is a mandatory baseline floor. Security engineers must actively probe for novel business-logic flaws, race conditions, and domain-specific abuse cases beyond these items.

---

## 1. Broken Access Control (OWASP A01:2021 / API1, API3, API5:2023)

- [ ] **1.1 BOLA / IDOR Prevention (API1:2023)**: Every database read, update, or delete query fetching a resource by ID (`/:id`) explicitly scopes the `WHERE` clause by the authenticated principal's `tenant_id` and `owner_id` (e.g., preventing Patient A from viewing Patient B's lab results, or Merchant A from refunding Merchant B's order).
- [ ] **1.2 Nested Object BOLA**: Verify that nested IDs inside request bodies (e.g., `POST /orders` containing `{ "shippingAddressId": "addr_other_user", "paymentMethodId": "pm_other_user" }`) are also validated for ownership by the current user/tenant.
- [ ] **1.3 Broken Function Level Authorization — BFLA (API5:2023)**: Administrative, moderation, and internal endpoints (`/admin/*`, `mutation { approvePermit }`) enforce explicit role/permission checks at the service layer, not just UI hiding or client-side router guards.
- [ ] **1.4 Mass Assignment / Broken Object Property Level Authorization (API3:2023)**: Incoming payloads are parsed through strict allowlist schemas (`z.strictObject()`) before reaching ORM `.create()` or `.update()`. Sensitive fields (`role`, `isAdmin`, `tenantId`, `accountBalance`, `emailVerified`, `subscriptionTier`) can never be overwritten by user input.
- [ ] **1.5 Field-Level Read Filtering**: API responses strip internal or privileged fields (`passwordHash`, `totpSecret`, `internalReviewerNotes`, `ssn`, `stripeCustomerSecret`) via explicit DTO serializers rather than returning raw ORM model instances.
- [ ] **1.6 Default-Deny Policy**: Access control frameworks default to **DENY** unless an explicit policy grants access; unauthenticated or untagged routes fail closed (`401`/`403`).
- [ ] **1.7 Headless CMS Local API Enforcement**: In Payload CMS 3.x (`payload.find/update/create/delete`), every Local API call invoked with user context explicitly sets `overrideAccess: false` and passes `user`. In Supabase, `service_role` keys are never used for user-scoped queries where RLS should apply.
- [ ] **1.8 Path Traversal & Local File Inclusion (CWE-22)**: Any file path constructed from user input normalizes the path (`path.resolve`), strips directory traversal sequences (`../`, `..\`, URL-encoded `%2e%2e%2f`, null bytes `\0`), and verifies `resolvedPath.startsWith(allowedBaseDir + path.sep)`.
- [ ] **1.9 Time-of-Check to Time-of-Use (TOCTOU) Race Protection**: Multi-request mutations on finite resources (redeeming a single-use gift card, transferring wallet funds, casting a single ballot, booking the last clinic slot) use atomic database operations (`UPDATE ... WHERE used_at IS NULL RETURNING *` or `SELECT ... FOR UPDATE`) rather than non-atomic `SELECT` followed by `UPDATE`.
- [ ] **1.10 CORS Strict Origin Allowlist**: `Access-Control-Allow-Origin` is never set to `*` alongside `Access-Control-Allow-Credentials: true`, nor does the server blindly reflect arbitrary `Origin` headers without validating against a strict static allowlist.

---

## 2. Injection & Input Handling (OWASP A03:2021)

- [ ] **2.1 SQL Injection (SQLi — CWE-89)**: 100% of database queries use parameterized prepared statements (`$1, $2`) or safe ORM query builders. Dynamic `ORDER BY` column names or `ASC`/`DESC` directions are validated against a hardcoded enum allowlist—never string-interpolated.
- [ ] **2.2 ORM / Filter Injection**: When using ORMs (Prisma, Drizzle, Mongoose, Sequelize), never pass unvalidated user JSON objects directly into `where:` filters (which allows attackers to inject operator objects like `{ "passwordResetToken": { "not": null } }` or `$ne: null`).
- [ ] **2.3 NoSQL Injection (CWE-943)**: Force scalar types (`z.string()`, `z.number()`) on all query parameters so attackers cannot pass MongoDB/CouchDB query operators (`$gt`, `$where`, `$regex`) via JSON bodies or query strings (`?token[$ne]=1`).
- [ ] **2.4 OS Command Injection (CWE-78)**: Never invoke shell interpreters (`child_process.exec`, `execSync`, `sh -c`, `os.system`). Use `execFile` or `spawn` with `shell: false` and pass arguments as an explicit string array validated against an allowlist.
- [ ] **2.5 Cross-Site Scripting — Stored, Reflected & DOM XSS (CWE-79)**:
  - Prohibit raw HTML injection sinks (`dangerouslySetInnerHTML`, `v-html`, `[innerHTML]`, `{@html}`, `element.innerHTML`) unless the string is sanitized immediately prior using **DOMPurify** with a strict tag/attribute allowlist.
  - Validate all user-supplied URLs rendered in `<a href={url}>` or `<img src={url}>` to allow only `https://`, `http://`, or `mailto:` schemes—explicitly blocking `javascript:`, `data:`, and `vbscript:` URIs.
- [ ] **2.6 Server-Side Template Injection — SSTI (CWE-1336)**: Never concatenate or pass user-controlled strings as the template source itself in Handlebars, EJS, Pug, Jinja2, Twig, or Liquid; pass user values strictly as template data context.
- [ ] **2.7 Prototype Pollution (CWE-1321)**: In JavaScript/TypeScript, forbid recursive object merge utilities that do not explicitly block `__proto__`, `constructor`, and `prototype` keys; use `Object.create(null)`, `Map`, or `z.strictObject()` validation.
- [ ] **2.8 Regular Expression Denial of Service — ReDoS (CWE-1333)**: Audit all custom regular expressions for catastrophic backtracking (nested quantifiers like `(a+)+`) and enforce strict maximum string length limits (`max(256)`) before regex execution.
- [ ] **2.9 GraphQL Introspection & Batching Abuse**: Disable GraphQL schema introspection in production (unless deliberately public), enforce query depth/complexity limits, and bound batch/alias counts per request to prevent DoS and brute-force attacks.
- [ ] **2.10 XML External Entity — XXE & Deserialization (CWE-611 / CWE-502)**: Disable DTDs and external entities (`resolveEntities: false`) in any XML/SVG/DOCX parser, and never deserialize untrusted native binary formats (`pickle`, PHP `unserialize`, Java `ObjectInputStream`, `node-serialize`).

---

## 3. Server-Side Request Forgery — SSRF (OWASP A10:2021 / API7:2023)

- [ ] **3.1 Outbound URL Allowlisting**: Whenever the backend fetches a URL influenced by user input (webhooks, avatar URL import, link previews, PDF rendering, OIDC discovery), validate the parsed URL protocol (`https:` only) and hostname against an explicit allowlist where possible.
- [ ] **3.2 Private IP & Cloud Metadata Blocking**:
  - Resolve the target hostname via DNS and block all connections to private, loopback, link-local, and cloud metadata IP ranges before connecting:
    - IPv4 Loopback & Local: `127.0.0.0/8`, `0.0.0.0/8`
    - RFC 1918 Private Networks: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`
    - **Cloud Metadata Endpoint**: `169.254.0.0/16` (specifically **`169.254.169.254`** AWS/GCP/Azure/DigitalOcean metadata and `fd00:ec2::254`)
    - IPv6 Loopback & Unique Local: `::1/128`, `fc00::/7`, `fe80::/10`, IPv4-mapped IPv6 (`::ffff:127.0.0.1`, `::ffff:169.254.169.254`).
- [ ] **3.3 DNS Rebinding & Redirect Bypass Protection**:
  - Pin the validated resolved IP address directly in the custom HTTP agent/socket connector so a second DNS lookup cannot rebind to `127.0.0.1` or `169.254.169.254`.
  - Disable automatic HTTP redirects (`redirect: 'error'` or `maxRedirects: 0`), or re-validate every `Location` redirect target through the exact same SSRF IP filter.
- [ ] **3.4 Cloud IMDSv2 Enforcement**: On cloud VMs (AWS EC2, etc.), enforce Instance Metadata Service v2 (IMDSv2) with hop-limit `1` (`HttpPutResponseHopLimit = 1`) so simple GET-based SSRF cannot steal IAM role credentials.

---

## 4. Authentication & Session Hardening (OWASP A07:2021 / NIST SP 800-63B)

- [ ] **4.1 Hardened Cookie Prefix & Attributes**:
  - Name session cookies with the **`__Host-`** prefix (e.g., `__Host-session_id`), which forces browsers to enforce: `Secure` flag required, `Path=/`, and **no `Domain` attribute** (preventing subdomain cookie injection attacks!).
  - Set `HttpOnly; Secure; SameSite=Lax` (or `SameSite=Strict` for high-assurance admin/banking portals).
- [ ] **4.2 Zero Sensitive Tokens in `localStorage` / `sessionStorage`**: Never store session tokens, long-lived JWTs, or refresh tokens in `window.localStorage` or `sessionStorage`, where any single XSS vulnerability or malicious npm script can exfiltrate them globally.
- [ ] **4.3 Cross-Site Request Forgery (CSRF) Defense**:
  - For cookie-authenticated endpoints, reject state-mutating requests via `GET`.
  - Enforce **Cryptographic CSRF Tokens** (Synchronizer Token Pattern or HMAC-signed Double-Submit Cookie bound to the user's session ID) AND verify the `Origin` / `Sec-Fetch-Site` request headers on all `POST`, `PUT`, `PATCH`, and `DELETE` endpoints.
- [ ] **4.4 Session Fixation & Revocation**:
  - Regenerate a brand-new session ID immediately upon login, privilege elevation, or password change.
  - Immediately invalidate all active sessions (and revoke refresh token families) upon password reset, password change, MFA disabling, or explicit "Sign out of all devices".
- [ ] **4.5 OAuth 2.1 / OIDC Security (`state` + `PKCE`)**:
  - Enforce **PKCE (`S256` code challenge)** and a cryptographically random **`state`** parameter bound to the browser cookie on all OAuth authorization code flows.
  - Validate `redirect_uri` against an exact string match allowlist (no wildcard path/subdomain open redirects!).
  - Do not automatically link an unverified OAuth social account to an existing email/password account without proving email ownership (prevents Pre-Account Takeover).
- [ ] **4.6 Account Enumeration Prevention**: Login, registration, and password-reset endpoints return identical generic messages (`"If an account exists for that email, a reset link has been sent"`) and normalized response timings (performing a dummy Argon2id hash when a username is not found).
- [ ] **4.7 Brute-Force & Credential Stuffing Protection**: Enforce progressive rate limiting and exponential lockouts keyed by both `(ip_address)` and `(normalized_email_or_username)` on login, OTP/MFA verification, and password reset endpoints.
- [ ] **4.8 Password Reset Token Security**: Generate password reset tokens with at least 256 bits of entropy, store only their **SHA-256 hash** in the database, expire them within `15–30 minutes`, enforce single-use consumption, and never leak the reset token to third parties via `Referer` headers (`Referrer-Policy: no-referrer` on the reset page).
- [ ] **4.9 Step-Up Authentication (Re-Auth)**: Require fresh password or WebAuthn/MFA verification before executing high-risk actions (changing email/password, exporting bulk EHR/financial data, adding payout bank accounts, generating API keys).

---

## 5. HTTP Security Headers & Browser Sandboxing (OWASP A05:2021)

Every HTML and API response must include hardened security headers:

- [ ] **5.1 Strict Content-Security-Policy (CSP)**:
  - Use cryptographic **per-request nonces** (`'nonce-<random>'`) or hashes alongside `'strict-dynamic'` instead of `'unsafe-inline'` or `'unsafe-eval'`:
    ```http
    Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-rAnd0m123' 'strict-dynamic'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self'; font-src 'self'; object-src 'none'; base-uri 'self'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests;
    ```
- [ ] **5.2 HTTP Strict Transport Security (HSTS)**:
  - `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`
- [ ] **5.3 MIME-Sniffing Prevention**:
  - `X-Content-Type-Options: nosniff`
- [ ] **5.4 Clickjacking Prevention**:
  - `frame-ancestors 'none'` (in CSP) and `X-Frame-Options: DENY`
- [ ] **5.5 Referrer Leakage Control**:
  - `Referrer-Policy: strict-origin-when-cross-origin` (or `no-referrer` on sensitive token/clinical/financial pages)
- [ ] **5.6 Browser Feature Restrictions (`Permissions-Policy`)**:
  - `Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=(), interest-cohort=()` (enable specific capabilities only on origins that explicitly require them)
- [ ] **5.7 Cross-Origin Isolation Headers**:
  - `Cross-Origin-Opener-Policy: same-origin`
  - `Cross-Origin-Resource-Policy: same-origin` (or `same-site`)
- [ ] **5.8 Cache-Control on Authenticated Responses**:
  - Set `Cache-Control: no-store, no-cache, must-revalidate, private` on all endpoints returning authenticated user data, PII, or tokens so CDN/shared proxies never cache private responses.

---

## 6. File Upload & Media Processing Security

- [ ] **6.1 Strict File Size & Count Limits**: Enforce hard byte-size limits at the reverse proxy / multipart parser stream level before buffering in memory.
- [ ] **6.2 Magic Byte (File Signature) Validation**: Never trust the client-supplied `Content-Type` header or filename extension alone. Inspect the actual file magic bytes (`file-type` library) against an explicit allowlist (`image/jpeg`, `image/png`, `image/webp`, `application/pdf`).
- [ ] **6.3 SVG & Active Content Prohibition**: Reject `.svg`, `.html`, `.xml`, `.swf`, and `.exe` uploads from untrusted users (as SVGs can contain embedded `<script>` XSS and XXE payloads).
- [ ] **6.4 Image Re-Encoding & EXIF Stripping**: Pass uploaded raster images through `sharp` / `libvips` to strip EXIF metadata (which often leaks GPS home coordinates of citizens/patients!) and re-encode pixels cleanly, neutralizing polyglot payload exploits.
- [ ] **6.5 Randomized Filenames & Isolated Storage Domain**:
  - Generate a random UUID filename (`01J9X2K8M.webp`); never use the user-provided filename on disk or in S3 keys.
  - Serve user-uploaded files from a **separate sandboxed domain** (e.g., `cdn-usercontent.example-files.com`, never the main app domain holding session cookies) with `Content-Disposition: attachment` for non-image documents and `X-Content-Type-Options: nosniff`.
- [ ] **6.6 Malware Scanning & Presigned URLs**: For document portals (Healthcare lab PDFs, Civic permit attachments, FinTech KYC uploads), scan uploads asynchronously via ClamAV before marking them available, and serve private files exclusively via short-lived (`60s`) presigned S3/R2 URLs after verifying user authorization.

---

## 7. WebSocket & Real-Time Channel Security (RFC 6455 / ASVS V13.4)

- [ ] **7.1 Cross-Site WebSocket Hijacking (CSWSH) Origin Validation**:
  - Standard CORS rules do not block cross-origin `wss://` upgrade handshakes, and browser cookies may be attached automatically during the `101 Switching Protocols` request.
  - Explicitly validate the `Origin` header on the WebSocket upgrade handshake against a strict allowlist of trusted origins, rejecting unknown origins with `403 Forbidden` before upgrading.
- [ ] **7.2 Per-Message Schema Validation & Room Authorization**:
  - Validate every inbound WebSocket message frame using a strict discriminated union schema (`z.discriminatedUnion('type', [...])`) and enforce a hard `maxPayload` byte limit (e.g., `64 KiB`) to prevent memory exhaustion.
  - Verify tenant and object-level authorization (`tenant_id`, `patient_id`, `order_id`) whenever a client attempts to subscribe to or publish on a real-time room/channel.
- [ ] **7.3 Continuous Token Re-Validation & Active Revocation**:
  - Do not authenticate only once at socket connection time and allow the socket to live forever after logout or token expiry.
  - Periodically re-verify session/token validity on active connections and immediately close sockets (`1008 Policy Violation`) when a user's session is revoked, password changes, or tenant permissions are downgraded.
