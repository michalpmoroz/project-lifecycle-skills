---
name: standards-omni-role-review
description: >-
  Enforce the Multi-Perspective 360-Degree Review Protocol on every project deliverable, feature specification, backlog refinement, architecture proposal, and Pull Request across all 14 specialized engineering, design, product, legal, security, and governance roles.
---

# Omni-Role 360-Degree Review Standards (`standards-omni-role-review`)

## Mission
Eliminate architectural blind spots, unstated product assumptions, accessibility regressions, security vulnerabilities, and legal exposure by subjecting every feature specification, task refinement, ADR, and Pull Request to a **mandatory 14-perspective 360-Degree Peer Review Gate**.

No feature backlog item may be marked `Ready for Dev`, and no Pull Request may be merged into `main`, until every applicable specialist role dimension is explicitly verified.

---

## Activation & Cross-Linking Protocol

Whenever `standards-omni-role-review` is activated:
1. **Load Project Extensions (Rule C)**:
   - Check `<workspace>/.agents/extensions/standards-omni-role-review.md` for project-specific compliance gates, custom domain review dimensions, or regulatory sign-off requirements.
2. **Load Living Documentation & Helper Checklists (Rule B & Rule D)**:
   - Cross-reference `<workspace>/docs/adr/`, `<workspace>/docs/domain/`, and the deep-dive checklists in `role-senior-code-reviewer/references/pr-code-review-checklist.md`, `design-engineering/references/ui-craft-and-ergonomics-checklist.md`, `role-qa-engineer/references/testing-pyramid-playwright-checklist.md`, `role-legal-counsel/references/gdpr-privacy-compliance-checklist.md`, and `role-seo-geo-specialist/references/technical-seo-geo-schema-checklist.md`.

> **Baseline Governance & Quality Standards (Rule G)**
> - **Accessibility & UX**: WCAG `2.2 Level AA`, APCA contrast, 8 Mandatory UI States, `48x48px` touch targets
> - **Performance**: Core Web Vitals (`LCP < 2.5s`, `INP < 200ms`, `CLS < 0.1`), Lighthouse Mobile `>= 90`
> - **Security & Privacy**: OWASP Top 10 (`2021`), OWASP API Security (`2023`), GDPR (`EU 2016/679`), EAA (`2025`)
> - **Testing & Code Quality**: TypeScript `5.7+` (`strict`), Playwright `1.49+`, `@axe-core/playwright` `4.10+`

---

## The 14-Role 360-Degree Audit Matrix

When refining a specification or reviewing a Pull Request, systematically evaluate all 14 specialist perspectives:

### 1. Product Owner (`role-product-owner`)
- [ ] **User Value & JTBD**: Does this directly solve a validated user problem without introducing scope creep?
- [ ] **Gherkin Completeness**: Are Happy Path, Empty/Boundary, and Error/Offline `Given/When/Then` scenarios satisfied?
- [ ] **Web Essentials Alignment**: Are relevant boilerplate requirements (`standards-web-essentials`) accounted for?

### 2. Business Analyst (`role-business-analyst`)
- [ ] **Ubiquitous Language**: Do all variable names, API fields, DB columns, and UI labels match `/docs/domain/glossary.md`?
- [ ] **State & Edge-Case Rigor**: Are cardinality (`0`, `1`, `N`), temporal (`NULL` dates, DST/timezones), missing data, and tie-breaking rules handled deterministically?

### 3. Software Architect (`role-software-architect`)
- [ ] **Module Boundaries**: Are frontend, backend, and pure shared domain layers cleanly decoupled with zero circular or forbidden imports?
- [ ] **ADR Compliance**: Does the implementation adhere to accepted decisions in `/docs/adr/` (or include a new ADR if introducing a new pattern)?

### 4. UI/UX Designer & Design Engineer (`role-ui-ux-designer`, `design-engineering`)
- [ ] **Zero AI Slop & Token Fidelity**: Uses semantic Penpot tokens (`OKLCH`/CSS variables); zero arbitrary hex codes or nested card clutter.
- [ ] **All 8 UI States**: Explicitly implements `Loading (Skeleton)`, `Empty`, `Partial`, `Ideal`, `Overflow / Long Text`, `Error (with Retry)`, `Offline`, and `Permission Denied`.
- [ ] **Mobile Ergonomics & Typography**: `>= 48x48px` tap targets, `env(safe-area-inset-*)`, `text-wrap: balance/pretty`, `tabular-nums` on numbers, and `font-size >= 16px` on mobile inputs.
- [ ] **Compositor Motion**: Animations touch only `transform` and `opacity` (`150ms–250ms`) and respect `prefers-reduced-motion: reduce`.

### 5. Frontend & Mobile Engineer (`role-frontend-developer`, `role-mobile-developer`)
- [ ] **Viewport Verification**: Verified at `390x844` mobile viewport first before tablet (`820px`) and desktop (`1280px+`).
- [ ] **State & Lifecycle Cleanliness**: Zero stale closures, uncancelled in-flight fetches (`AbortController`), or leaked event listeners/subscriptions.
- [ ] **Offline & PWA Readiness**: Handles intermittent mobile connectivity gracefully without losing user input.

### 6. Backend Developer (`role-backend-developer`)
- [ ] **Runtime Boundary Validation**: 100% of inbound body, query, path, and header inputs validated via strict Zod/Valibot schemas.
- [ ] **Idempotency & Concurrency**: Multi-step writes use atomic DB transactions; mutations guard against TOCTOU race conditions and duplicate submissions.
- [ ] **Database Hygiene**: Migrations follow `expand-and-contract` backward compatibility; zero N+1 queries; collection endpoints enforce bounded pagination.

### 7. QA & Test Automation Engineer (`role-qa-engineer`)
- [ ] **Testing Pyramid Coverage**: Pure logic tested via fast unit tests (Vitest/Jest); boundaries via integration tests; critical flows via Playwright E2E.
- [ ] **Zero Flakiness & Semantic Locators**: Uses `getByRole` / `getByLabel` (zero brittle XPath/CSS selectors) and web-first assertions (`zero waitForTimeout`).
- [ ] **Automated A11y Scan**: `@axe-core/playwright` passes with zero WCAG 2.2 AA violations across interactive states.

### 8. Security Consultant (`role-security-consultant`)
- [ ] **AuthN & AuthZ (BOLA/IDOR)**: Every endpoint verifies both authentication and resource-level tenant/owner authorization.
- [ ] **Injection & XSS Defense**: Parameterized queries only; strict CSP; escaped/sanitized HTML and JSON-LD (`\u003c`).
- [ ] **Rate Limiting & Secrets**: Public/expensive endpoints are rate-limited; zero secrets or private keys in code or client bundles.

### 9. Performance Consultant (`role-performance-consultant`)
- [ ] **Core Web Vitals Budget**: `LCP < 2.5s`, `INP < 200ms`, `CLS < 0.1` (explicit image/skeleton dimensions, modern `AVIF`/`WebP` assets).
- [ ] **Bundle Discipline**: Heavy libraries (maps, charts, editors) are code-split / lazy-loaded off the initial critical path.

### 10. Legal Counsel & Privacy Engineer (`role-legal-counsel`)
- [ ] **GDPR & PII Scrubbing**: Every PII field has a documented Art. 6 lawful basis; zero PII (emails, tokens, IPs) leaked in URLs or logs (`pino`/Sentry redaction).
- [ ] **Cookie & Consent Parity**: Cookieless analytics by default, or strict opt-in with equal-prominence `Reject All` / `Accept All`.
- [ ] **Defamation & OSS Licenses**: Public claims link to primary institutional sources with legal status enums; dependencies are free of incompatible viral copyleft (`AGPL`/`SSPL`).

### 11. SEO & Geospatial Specialist (`role-seo-geo-specialist`)
- [ ] **Crawlability & Metadata**: Server-rendered `<title>`, `<meta description>`, self-referencing `<link rel="canonical">`, Open Graph `1200x630` image, and connected Schema.org JSON-LD.
- [ ] **Map-to-List Parity**: All geospatial entities plotted on interactive maps are also reachable via crawlable HTML directory links.

### 12. UX Writer & Copywriter (`role-copywriter`)
- [ ] **Clarity & Empathy**: Action-oriented `Verb + Noun` buttons, 3-part error messages (*what happened, why, how to fix*), and zero technical jargon.
- [ ] **i18n Readiness**: ICU pluralization and `Intl` formatting used; zero hardcoded string concatenation.

### 13. DevOps & SRE (`role-devops-engineer`)
- [ ] **CI/CD & Reproducibility**: Builds, lints, typechecks, and tests run deterministically in GitHub Actions; new environment variables are documented in `.env.example` and validated at startup.

### 14. Technical Archivist & Code Reviewer (`role-archivist`, `role-senior-code-reviewer`, `standards-code-style`)
- [ ] **Living Documentation Sync**: `/docs/adr/`, `/docs/domain/`, and `/docs/architecture/` are updated in the same PR and tagged with `skills: [...]` frontmatter.
- [ ] **Strict Type, Code Style & Lint Health**: Adheres to `standards-code-style` (formatting, naming conventions, complexity budgets, 5-tier import order); zero `any`, `@ts-ignore`, silent `catch {}`, or leftover debug code; Conventional Commits with zero AI co-author trailers.

---

## Continuous Evolution & Self-Improvement Mandate
- **Baseline Floor, Not a Ceiling**: This 14-role review matrix is the mandatory baseline floor. When a defect escapes to `main` or production, analyze which role perspective missed it and strengthen both the project extension (`.agents/extensions/standards-omni-role-review.md`) and the corresponding base role checklist.
