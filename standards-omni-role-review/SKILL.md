---
name: standards-omni-role-review
description: >-
  Enforce the Multi-Perspective 360-Degree Review Protocol on every project deliverable, feature specification, task refinement, and Pull Request across all specialized roles (Product Owner, Business Analyst, Software Architect, UI/UX Designer, Design Engineer, Frontend Developer, Backend Developer, QA Engineer, Security Consultant, Performance Consultant, Copywriter, DevOps Engineer, Legal Counsel, Archivist, SEO/GEO Specialist, and Code Reviewer).
---

# Omni-Role 360-Degree Review Standards (`standards-omni-role-review`)

## Mission
Eliminate blind spots, unstated assumptions, and post-launch regressions by submitting every issue, task specification, architectural proposal, and Pull Request to a **rigorous multi-role peer review protocol**.

No feature or code change may be merged without satisfying the criteria of each relevant specialist perspective.

---

## The 360-Degree Review Checklist by Perspective

When refining an issue or reviewing a Pull Request, the team / agent must verify compliance across each domain:

### 1. Product Owner & Business Analyst (`role-product-owner`, `role-business-analyst`)
- [ ] Does this directly deliver user value to the target audience (end users/contributors/administrators)?
- [ ] Are functional and non-functional requirements explicitly met with Gherkin acceptance criteria?
- [ ] Are edge cases defined (e.g. entity with missing category, zero results, boundary conditions)?

### 2. Legal Counsel & Compliance (`role-legal-counsel`)
- [ ] Is data privacy and protection (GDPR/compliance) respected (no private phone numbers, home addresses, or national identification numbers exposed)?
- [ ] Is defamation liability and legal dispute exposure mitigated with verifiable source citations?
- [ ] Does any tracking or form submission comply with privacy and cookie consent mandates?

### 3. UI/UX Designer & Design Engineer (`role-ui-ux-designer`, `design-engineering`)
- [ ] Do all interactive controls meet the minimum **48x48px** touch-target threshold?
- [ ] Are hardware insets respected via safe area variables (`env(safe-area-inset-*)`)?
- [ ] Are tabular figures applied to counters, dates, and metrics (`tabular-nums`)?
- [ ] Is the design free of generic AI slop, adhering strictly to established Penpot tokens?

### 4. Technical Architect & Backend Developer (`role-software-architect`, `role-backend-developer`)
- [ ] Is the schema normalized and backward-compatible (no breaking DB migrations)?
- [ ] Are all inbound API endpoints strictly validated with runtime Zod schemas?
- [ ] Is the codebase 100% English in logic, types, variables, and comments?

### 5. Frontend & Mobile Engineer (`role-frontend-developer`, `role-mobile-developer`)
- [ ] Is the viewport tested at **390px** mobile width before wider desktop screens?
- [ ] Are animations compositor-only (`transform`, `opacity`) and snappy (150-250ms)?
- [ ] Does the page function gracefully on slow mobile networks or offline?

### 6. QA & Test Automation (`role-qa-engineer`)
- [ ] Do automated unit tests cover positive, negative, and boundary conditions?
- [ ] Are mobile viewport E2E tests configured (Playwright)?

### 7. Security Consultant (`role-security-consultant`)
- [ ] Are zero unauthenticated writes permitted?
- [ ] Is Content Security Policy (CSP) enforced and all user/external inputs sanitized against XSS/SQLi?
- [ ] Are rate limits configured on search and geocoding endpoints?

### 8. Performance Consultant (`role-performance-consultant`)
- [ ] Are Core Web Vitals protected (LCP < 2.5s, INP < 200ms, CLS < 0.1)?
- [ ] Are images modern-format (WebP/AVIF) with explicit width/height dimensions?

### 9. SEO & Geospatial Specialist (`role-seo-geo-specialist`)
- [ ] Are canonical tags, semantic Schema.org JSON-LD, and Open Graph tags present?
- [ ] Is geographic and navigational taxonomy cleanly discoverable by search crawlers?

### 10. Copywriter & UX Writer (`role-copywriter`)
- [ ] Is all user-facing copy natural, neutral, respectful, and free of technical jargon?
- [ ] Are empty states and error messages clear, actionable, and helpful?

### 11. Technical Archivist (`role-archivist`)
- [ ] Are architecture decisions recorded in `/docs/adr/`?
- [ ] Are framework-agnostic domain models updated in `/docs/domain/`?
- [ ] Is all critical project knowledge persisted to the local repository, not ephemeral cache?
