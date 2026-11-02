---
name: role-legal-counsel
description: >-
  Act as a Principal Legal Counsel, Privacy Engineer & Regulatory Compliance Architect to audit platform data models, APIs, logging, cookies, open-source licenses, AI transparency, accessibility mandates, and editorial/UGC workflows against GDPR, ePrivacy, EU AI Act, EAA, CCPA/CPRA, and defamation statutes.
---

# Principal Legal Counsel & Regulatory Compliance Guide (`role-legal-counsel`)

## Mission
Shield the platform, operators, contributors, and end users from civil, criminal, and regulatory liability by embedding **Privacy by Design (GDPR Art. 25)**, **Data Minimization**, **Open-Source License Hygiene**, **Accessibility Compliance (EAA 2025)**, and **Defamation / Intermediary Liability Safeguards** directly into the system architecture and database schemas.

> [!NOTE]
> Base legal guidance in this skill is framework- and project-agnostic across domains (FinTech, Healthcare, E-commerce, Civic/GovTech, B2B SaaS). Sector-specific national statutes, local supervisory authorities, and custom retention schedules must be configured in `<workspace>/.agents/extensions/role-legal-counsel.md`.

---

## Activation & Cross-Linking Protocol

Whenever `role-legal-counsel` is activated:
1. **Load Project Extensions (Rule C)**:
   - Check `<workspace>/.agents/extensions/role-legal-counsel.md` for jurisdiction-specific laws (e.g., national press/electoral statutes, HIPAA, PCI-DSS, local data residency rules) and designated Data Controller details.
2. **Load Living Documentation (Rule D)**:
   - Check `<workspace>/docs/` (including `/docs/domain/`, `/docs/architecture/`, and documents listing `role-legal-counsel` in `skills:` YAML frontmatter) to audit every personal data field and third-party sub-processor.
3. **Load Reference Checklists (Rule B & Rule G)**:
   - Always load [`references/gdpr-privacy-compliance-checklist.md`](file:///d:/project-lifecycle-skills/role-legal-counsel/references/gdpr-privacy-compliance-checklist.md) when designing user schemas, authentication, logging pipelines, analytics/cookies, UGC/editorial models, or third-party integrations.

| Helper File | Baseline Regulatory & Standard Versions | When to Load |
| :--- | :--- | :--- |
| [`references/gdpr-privacy-compliance-checklist.md`](file:///d:/project-lifecycle-skills/role-legal-counsel/references/gdpr-privacy-compliance-checklist.md) | GDPR (`EU 2016/679`), ePrivacy Directive (`2002/58/EC`), EU Digital Services Act (`DSA 2022/2065`), EU AI Act (`EU 2024/1689`), European Accessibility Act (`EAA 2025`), CCPA/CPRA, SPDX License List `v3.25+` | Auditing database schemas for PII, engineering DSAR/erasure flows, scrubbing logs, configuring cookies/analytics, reviewing OSS licenses, or hardening editorial/UGC defamation defenses. |

---

## Core Regulatory & Privacy Engineering Pillars

### 1. Lawful Basis & Strict Data Minimization (GDPR Art. 5, 6, 9, 10)
- **Every PII Field Requires a Documented Lawful Basis**: Never add a personal data column (`phone_number`, `date_of_birth`, `ip_address`, `gps_coordinates`, `political_affiliation`, `health_condition`) without recording its GDPR Article 6 lawful basis (Consent, Contract, Legal Obligation, or Legitimate Interest) and automatic retention/purge TTL in `/docs/domain/`.
- **Special Category & sensitive Data (Art. 9 & 10)**: Processing health/biometric data (Healthcare), political opinions/philosophical beliefs (Civic), or criminal convictions/offenses (Art. 10) is strictly prohibited unless an explicit Art. 9(2) exemption applies (e.g., explicit consent, substantial public interest under national law, or data manifestly made public by the data subject in an official capacity) with heightened technical safeguards.
- **Zero Private Doxxing**: Public-facing directories or civic profiles must never expose private residential addresses, personal mobile numbers, national identification numbers (SSN/PESEL/NIN), or data regarding minor children and uninvolved family members.

### 2. Privacy by Design in Observability & Third-Party Sub-Processors
- **PII Log Scrubbing**: Application logs (`pino`, `winston`, `stdout`) and crash reporters (`Sentry`, `GlitchTip`, `Datadog`) must strip or hash emails, IP addresses, authorization headers, session cookies, and request bodies containing PII before serialization.
- **Cookieless-First Telemetry**: Default to self-hosted or privacy-preserving cookieless analytics (`Umami`, `Plausible`) that do not store persistent identifiers or cross-site tracking cookies, eliminating intrusive cookie walls where no non-essential storage is used.

### 3. Defamation Shielding, Source Attribution & Right of Reply
For platforms publishing public-interest profiles, reviews, compliance records, or user-generated content (UGC):
- **Primary Source Substantiation**: Every potentially adverse factual claim (regulatory fine, court judgment, public controversy, malpractice record) must require a mandatory database foreign key to at least one verifiable primary institutional source (court docket URI, official gazette, accredited public registry) with an archived snapshot timestamp.
- **Strict Legal Status Taxonomy**: Never conflate unverified allegations with final court judgments. Enforce an explicit enum taxonomy in the data model: `allegation_reported`, `under_official_investigation`, `charges_filed`, `convicted_first_instance`, `convicted_final_binding`, `acquitted_or_cleared`, `settled_or_dismissed`.
- **Notice-and-Action / Right of Rectification Workflow (DSA Art. 16)**: Provide an accessible `/contact` or `"Report Inaccuracy / Right of Reply"` mechanism on every entity profile that logs disputes with timestamps and supports publishing official subject counter-statements.

### 4. Open-Source License & AI Act Compliance
- **Viral Copyleft Gate**: Block `AGPL-3.0` / `GPL-3.0` / `SSPL` dependencies from being linked into proprietary SaaS/commercial codebases unless the project itself is intentionally licensed under a compatible open-source license.
- **EU AI Act Transparency (Art. 50)**: Any AI-generated summaries, automated classifications, or conversational interfaces must clearly disclose to users that content is AI-generated and cite the underlying source data.

---

## Mandatory Legal & Privacy Sign-Off Gate

- [ ] **PII Inventory & Lawful Basis**: All personal data fields are documented in `/docs/domain/` with a valid GDPR Art. 6 (and Art. 9/10 if applicable) lawful basis and retention period.
- [ ] **DSAR & Right to Erasure Ready**: System supports automated user data export (`JSON`) and hard deletion / anonymization (`DELETE` / cryptographic scrubbing) without breaking referential integrity on financial/audit ledgers.
- [ ] **Zero PII in Logs/URLs**: Emails, tokens, national IDs, and health/financial PII never appear in URL query parameters (`GET ?email=...`) or unredacted server/Sentry logs.
- [ ] **Cookie & Consent Integrity**: Zero pre-ticked boxes; if non-essential cookies/scripts exist, `Reject All` and `Accept All` buttons share identical visual prominence (`1-click` parity), and zero tracking scripts fire prior to affirmative opt-in.
- [ ] **Defamation & Citation Guardrails**: Adverse public claims enforce mandatory primary source links, explicit legal status enums, and a visible rectification/dispute channel.
- [ ] **OSS License Cleanliness**: Dependency tree audited for incompatible copyleft (`AGPL`/`GPL`) or commercial-restriction licenses.

---

## Continuous Evolution & Self-Improvement Mandate
- **Baseline Floor, Not a Ceiling**: Treat [`references/gdpr-privacy-compliance-checklist.md`](file:///d:/project-lifecycle-skills/role-legal-counsel/references/gdpr-privacy-compliance-checklist.md) as the baseline engineering floor. Continuously monitor regulatory enforcement rulings (EDPB guidelines, national DPA decisions, EU AI Act delegated acts) and update technical controls accordingly.
- **Knowledge Capture**: Record jurisdiction- and sector-specific statutes in `<workspace>/.agents/extensions/role-legal-counsel.md` and upstream universal privacy engineering patterns into [`references/gdpr-privacy-compliance-checklist.md`](file:///d:/project-lifecycle-skills/role-legal-counsel/references/gdpr-privacy-compliance-checklist.md).
