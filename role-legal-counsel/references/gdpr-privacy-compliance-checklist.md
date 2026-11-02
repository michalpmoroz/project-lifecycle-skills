# GDPR, Privacy Engineering & Regulatory Compliance Checklist

> **Baseline Regulatory & Standard Versions (Rule G)**
> - **Data Protection & Privacy**: EU General Data Protection Regulation (`GDPR` Regulation `(EU) 2016/679`), UK GDPR, California Consumer Privacy Act / CPRA (`Cal. Civ. Code § 1798.100+`)
> - **Electronic Communications & Cookies**: ePrivacy Directive (`2002/58/EC` as amended by `2009/136/EC`) & EDPB Guidelines `2/2023`
> - **Platform & Intermediary Liability**: EU Digital Services Act (`DSA` Regulation `(EU) 2022/2065`)
> - **Artificial Intelligence**: EU AI Act (`Regulation (EU) 2024/1689` — Art. 50 Transparency & High-Risk Annex III rules)
> - **Digital Accessibility Mandate**: European Accessibility Act (`EAA` Directive `(EU) 2019/882`, enforceable June `2025` via EN `301 549` / WCAG `2.2 AA`)
> - **Open-Source Licensing**: SPDX License List `v3.25+`

---

## 1. Lawful Basis & Special Category Data Engineering (GDPR Art. 6, 9, 10)

Every database table or document collection storing personal data must map each attribute to one of the six lawful bases under **GDPR Article 6(1)**:

| Lawful Basis (Art. 6) | Engineering & Product Requirements | Multi-Domain Examples |
| :--- | :--- | :--- |
| **`6(1)(a)` Consent** | Freely given, specific, informed, unambiguous opt-in; granular per purpose; must store `consented_at`, `policy_version`, and support 1-click withdrawal (`withdrawn_at`). | Marketing newsletters (E-commerce), optional health wearables sync (Healthcare), non-essential analytics. |
| **`6(1)(b)` Contract** | Strictly necessary to perform a contract with the user. Cannot be bundled with marketing or behavioral profiling. | Shipping address & payment token (E-commerce), account credentials & workspace membership (B2B SaaS). |
| **`6(1)(c)` Legal Obligation** | Required by statutory law (e.g., tax/accounting retention, AML/KYC). | Retaining invoice ledgers for 5–10 years (FinTech), mandatory clinical audit logs (Healthcare). |
| **`6(1)(f)` Legitimate Interest** | Requires a documented **Legitimate Interest Assessment (LIA)** balancing necessity against data subject rights, plus an accessible **Right to Object (Art. 21)** workflow. | Security fraud/rate-limit IP hashing, aggregating official public-office records of elected officials or corporate directors (Civic/GovTech). |

### Special Categories of Data (Art. 9) & Criminal Records (Art. 10)
- **High-Risk Attributes**: Health/medical records, genetic/biometric data, racial/ethnic origin, political opinions, religious beliefs, trade union membership, sexual orientation (Art. 9), and criminal convictions/offenses (Art. 10).
- **Mandatory Engineering Controls**:
  - Field-level encryption at rest (or envelope encryption via KMS) and strict Row-Level Security (RLS) / RBAC audit logging on every read.
  - For public-interest civic/editorial registries: restrict processing strictly to facts manifestly made public by the figure or published in official institutional registries, and never infer sensitive traits via automated profiling.

---

## 2. Engineering Data Subject Rights (GDPR Art. 15–22 & CCPA)

Never design a schema where fulfilling a user's privacy request requires manual SQL surgery across 30 tables.

### Right of Access & Portability (Art. 15 & 20 — DSAR Export)
- Provide an automated service function (`exportUserData(userId)`) that compiles all personal data, profile settings, consent logs, and activity history tied to `userId` into a clean, machine-readable `JSON` archive within statutory deadlines (`<= 30 days`, ideally instant self-service).

### Right to Erasure / "Right to be Forgotten" (Art. 17)
- **The Foreign-Key vs. Audit Ledger Dilemma**:
  - Simply running `DELETE FROM users WHERE id = $1` often fails or corrupts financial invoices, medical audit trails, or immutable public contributions that must be retained under `Art. 6(1)(c)` (Legal Obligation) or `Art. 17(3)`.
- **Staff Engineering Pattern — PII Separation & Tombstone Anonymization**:
  1. Store mutable PII (`email`, `full_name`, `phone`, `avatar_url`, `oauth_tokens`) in an isolated `user_identities` / `user_pii` table or clearly demarcated columns.
  2. On account deletion:
     - Hard-delete active sessions, OAuth tokens, saved addresses, and notification preferences.
     - Scrub PII fields on the core user record (`email = 'deleted-' || id || '@erased.invalid'`, `full_name = '[Deleted User]'`, `phone = NULL`, `ip_hash = NULL`, `deleted_at = NOW()`).
     - Retain strictly required tax/invoice records under statutory retention locks, and purge them automatically once the statutory window expires.
     - Use **Cryptographic Erasure** (destroying the per-user encryption key) if immutable event streams/backups contain encrypted user payloads.

---

## 3. Privacy by Design & Default (Art. 25): PII Scrubbing & Minimization

### Zero PII Leakage in Logs, Crash Reporters & URLs
- **Never Put PII in URLs**: Never pass `email`, `token`, `password`, `ssn`, or `phone` in URL query strings (`/reset-password?email=jane@example.com`). URLs are recorded in plaintext across browser history, CDN access logs, corporate proxies, and `Referer` headers.
- **Structured Logger Redaction (`pino` / `Sentry`)**:
  Configure automatic redaction paths at the logger root so developers cannot accidentally log sensitive fields:

```typescript
// src/server/lib/logger.ts
import pino from 'pino';

export const logger = pino({
  level: process.env.LOG_LEVEL ?? 'info',
  redact: {
    paths: [
      'req.headers.authorization',
      'req.headers.cookie',
      '*.password',
      '*.passwordHash',
      '*.email',
      '*.phone',
      '*.nationalId',
      '*.creditCard',
      '*.clientIp',
      '*.accessToken',
      '*.refreshToken',
    ],
    censor: '[REDACTED_PII]',
  },
});
```

- **IP Address Minimization**: Do not store raw user IP addresses in database tables for analytics or rate limiting. Truncate IPv4/IPv6 addresses or compute a daily rotating salted HMAC (`HMAC_SHA256(ip, daily_rotating_salt)`) so abuse prevention works without storing persistent personal identifiers.

---

## 4. Cookie, Storage & Consent Engineering (ePrivacy & EDPB)

### Cookieless-First Architecture
- Prefer **cookieless analytics** (e.g., self-hosted `Umami` or `Plausible`) and strictly functional session/CSRF cookies (`__Host-session`, `HttpOnly`, `Secure`, `SameSite=Lax`).
- Under ePrivacy Directive Art. 5(3), **strictly necessary** cookies (authentication session, shopping cart state, security CSRF tokens, user's chosen language/theme preference) do **not** require a consent banner, provided they are disclosed in the Privacy Policy.

### When Consent Banners Are Required (Non-Essential Storage / Third-Party Trackers)
If any non-essential analytics, advertising pixels, third-party video embeds (YouTube/Vimeo), or marketing cookies are used:
1. **Zero Execution Before Consent**: No non-essential cookie, `localStorage` tracker, or third-party script may load or fire network requests until the user actively clicks `Accept`.
2. **No Dark Patterns (EDPB Guidelines 3/2022)**:
   - **`Reject All` Parity**: The `Reject All` (or `Only Necessary`) button must sit on the **exact same first layer** as `Accept All`, with identical button size, font weight, and visual contrast. Hiding rejection behind a secondary *"Manage Preferences"* link is unlawful.
   - **Zero Pre-Ticked Boxes**: All optional category toggles (`Analytics`, `Marketing`) must default to `OFF` (`aria-checked="false"`).
   - **Easy Withdrawal**: Provide a persistent footer link (`"Privacy & Cookie Settings"`) allowing users to revoke consent in 1 click at any time.

---

## 5. Defamation, Editorial Integrity & UGC Intermediary Shielding

For platforms displaying information about individuals, companies, professionals, or hosting user reviews/contributions:

### Defamation Defense Architecture
- **Source-Backed Schema Constraint**:
  - Database tables storing public controversies, disciplinary actions, legal proceedings, or fact-checks must enforce a `NOT NULL` constraint on `primary_source_url`, `source_publisher`, `source_published_date`, and `verification_status`.
- **Mandatory Legal Status Taxonomy**:
  - Clearly distinguish allegations from proven facts in both the database enum and UI badge rendering:
    - `allegation_or_report` -> Neutral informational badge (*"Reported in media / Under review"*)
    - `official_investigation` -> Warning badge (*"Ongoing administrative/judicial inquiry"*)
    - `final_court_judgment` -> Verified badge (*"Confirmed by final binding court ruling [Case Ref]"*)
    - `acquitted_or_dismissed` -> Cleared badge (*"Cleared of allegations / Case dismissed"*)
- **Neutral, Non-Defamatory Copy**:
  - Never use conclusory criminal labels (*"corrupt"*, *"fraudster"*, *"malpractice guilty"*) unless directly quoting a final binding criminal conviction. State objective, verifiable institutional facts (*"On [Date], [Authority] issued decision [Ref] regarding..."*).

### DSA Notice-and-Action & Right of Reply (DSA Art. 16)
- Every public profile or user-generated item must include an accessible `"Report Inaccuracy / Legal Notice"` action collecting:
  1. Specific URL and statement contested.
  2. Reason for notice (factual error, outdated court outcome, privacy violation, defamation, copyright).
  3. Supporting documentary evidence link/upload.
  4. Contact email and good-faith declaration.

---

## 6. Open-Source License Compliance (SPDX Audit)

Before adding any npm / PyPI / Go / Cargo dependency, verify its SPDX license compatibility with the project's distribution model:

| License Category | SPDX Identifiers | Commercial / SaaS Risk Level & Rule |
| :--- | :--- | :--- |
| **Permissive (Safe)** | `MIT`, `Apache-2.0`, `BSD-2-Clause`, `BSD-3-Clause`, `ISC`, `0BSD`, `Unlicense` | **Green (Approved)**: Safe for all open-source, commercial, and proprietary applications. Retain copyright notices in distributions. |
| **Weak Copyleft (File/Library Scoped)** | `MPL-2.0`, `LGPL-2.1`, `LGPL-3.0`, `EPL-2.0` | **Yellow (Conditional)**: Safe if dynamically linked or unmodified; modifications to the licensed library's own files must be shared under the same license. |
| **Strong / Network Copyleft (Viral)** | `GPL-2.0`, `GPL-3.0`, `AGPL-3.0`, `SSPL-1.0`, `Commons Clause`, `BUSL-1.1` | **Red (Restricted)**: `AGPL-3.0` requires releasing the **entire application's source code** if users interact with it over a network/API! `SSPL` and `BUSL` forbid commercial SaaS hosting. **Block in proprietary projects** unless an explicit dual-license exists. |

---

## 7. Essential Legal Pages & Regulatory Disclosures Checklist

Every production web application must ship with the following accessible, plain-language legal routes (cross-referenced with `standards-web-essentials`):
- [ ] **Privacy Policy (`/privacy`)**: Identifies Data Controller & DPO contact, lists exact data categories collected, lawful bases (Art. 6), sub-processors (hosting, email, analytics) with regions/SCCs, retention periods, Data Subject Rights (Access, Rectification, Erasure, Portability, Objection), and right to lodge a complaint with the supervisory authority.
- [ ] **Terms of Service (`/terms`)**: Defines scope of service, acceptable use policy, intellectual property licensing, limitation of liability, disclaimer of warranties (e.g., *"Informational only — does not constitute medical/legal/financial advice"*), and governing law.
- [ ] **Accessibility Statement (`/accessibility` — EAA 2025)**: Declares WCAG 2.2 AA conformance status, known limitations, and dedicated accessibility feedback contact channel.
- [ ] **AI Transparency Notice (EU AI Act Art. 50)**: If LLMs or automated scoring/summaries are used, clearly labels AI-assisted outputs and explains human oversight and source verification mechanisms.
