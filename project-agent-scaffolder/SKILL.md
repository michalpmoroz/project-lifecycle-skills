---
name: project-agent-scaffolder
description: >-
  Scaffold hierarchical, directory-scoped agent instructions (AGENTS.md, GEMINI.md, .agents/rules/, and .agents/extensions/) across project subdirectories so AI coding agents automatically enforce local architecture, UI craft, testing, and security constraints.
---

# Project Agent Scaffolder & Scoped Rules Guide (`project-agent-scaffolder`)

## Mission
Generate clean, hierarchical, directory-scoped instruction files (`AGENTS.md`, `GEMINI.md`, `.agents/rules/`, and `.agents/extensions/`) across a target project's directory tree. This guarantees that any AI coding assistant (Antigravity, Cursor, Claude Code, GitHub Copilot, Windsurf) automatically loads the right role skills, framework helpers, and project-specific extensions whenever operating inside a specific directory.

---

## Activation & Cross-Linking Protocol

When executing `project-agent-scaffolder`:
1. **Inspect Existing Workspace Context**:
   - Check `<workspace>/package.json` (or language-equivalent manifest), `<workspace>/docs/adr/`, and `<workspace>/.agents/extensions/` to identify the exact tech stack and versions selected for the project.
2. **Enforce Governance Rules (`standards-skills-governance`)**:
   - Root and scoped `AGENTS.md` files in the target project may contain project-specific stack rules, while linking directly to the shared base skills and `<workspace>/.agents/extensions/<skill-name>.md` delta files.
   - Ensure all generated Markdown files use clean standard spaces (never corrupted box-drawing characters) and standard UTF-8 encoding.

---

## Scoped Instructions Architecture

Modern AI coding agents resolve `AGENTS.md` hierarchically from the repository root down to the active working directory. Scaffolding scoped `AGENTS.md` files keeps context focused and prevents token bloat while enforcing strict layer boundaries:

```text
<project-root>/
├── AGENTS.md                           # Global repository rules, lifecycle gates, skill routing table
├── .agents/
│   ├── rules/
│   │   ├── git-conventions.md          # Conventional Commits, branch naming, zero AI co-author rule
│   │   ├── mobile-ergonomics.md        # 48x48px touch targets, safe areas, 390px viewport baseline
│   │   └── security-baseline.md        # Secret scanning, input validation, OWASP guardrails
│   └── extensions/
│       ├── role-frontend-developer.md  # Project-specific UI overrides & design system conventions
│       ├── role-backend-developer.md   # Project-specific ORM, auth, and API error envelope rules
│       ├── role-qa-engineer.md         # Project-specific test fixtures, seed scripts, and E2E URLs
│       ├── role-legal-counsel.md       # Project-specific regulatory statutes (e.g., HIPAA, GDPR, PCI)
│       ├── role-seo-geo-specialist.md  # Project-specific sitemap partitions & Schema.org types
│       └── standards-code-style.md     # Project-specific formatter/linter & naming overrides
├── docs/
│   ├── README.md                       # Living documentation master index
│   ├── adr/                            # Architecture Decision Records
│   ├── domain/                         # Framework-agnostic ERD, glossary, and business rules
│   └── architecture/                   # System blueprints, API contracts, threat models
├── src/
│   ├── app/ (or frontend/ / web/)
│   │   └── AGENTS.md                   # Scoped Frontend/UI rules -> links to framework helper & Penpot tokens
│   ├── server/ (or api/ / backend/)
│   │   └── AGENTS.md                   # Scoped Backend/API rules -> links to DB, auth, and validation helpers
│   └── shared/ (or packages/core/)
│       └── AGENTS.md                   # Scoped Pure Domain/Types rules -> zero framework or I/O imports
└── tests/ (or e2e/)
    └── AGENTS.md                       # Scoped QA rules -> links to Playwright & testing pyramid checklist
```

---

## How Scoped `AGENTS.md` Files Link to Extensions & Framework Helpers

Every scoped `AGENTS.md` file must establish a **three-tier resolution chain**:
1. **Base Role Skill**: Identifies which core skill(s) govern the directory (e.g., `role-frontend-developer`, `design-engineering`).
2. **Framework / Technical Helper File**: Points directly to the exact technology helper file matching the project's stack (e.g., `role-frontend-developer/frameworks/react.md` or `role-qa-engineer/references/testing-pyramid-playwright-checklist.md`).
3. **Project Extension File**: Instructs the agent to load `<project-root>/.agents/extensions/<skill-name>.md` and relevant `<project-root>/docs/` files tagged with that skill in YAML frontmatter.

---

## Standard Scoped Templates

### 1. Root `<project-root>/AGENTS.md` Template

```markdown
# Project Engineering & AI Agent Governance (`AGENTS.md`)

## 1. Skill Resolution & Extension Protocol
Before planning or writing code in any directory, you MUST resolve context in this order:
1. **Scoped `AGENTS.md`**: Read the local `AGENTS.md` in the target subdirectory (`src/app/`, `src/server/`, `src/shared/`, `tests/`).
2. **Base Role Skill & Helper Files**: Load the applicable role skill (`role-*`, `design-engineering`, `standards-*`) and its specific `frameworks/` or `references/` helper file matching our tech stack.
3. **Project Extensions**: Read `.agents/extensions/<skill-name>.md` for project-specific overrides and domain rules.
4. **Living Documentation**: Check `/docs/README.md`, `/docs/adr/`, and `/docs/domain/` for documents tagged with your active skill in YAML frontmatter (`skills: [...]`).

## 2. Non-Negotiable Repository Invariants
- **Baseline Code Style & Static Analysis**: Enforce `standards-code-style` across all files (UTF-8, `LF`, 2-space indent, 100-char line width, 5-tier sorted imports, cyclomatic complexity `<= 10`, cognitive complexity `<= 15`, explicit unit suffixes).
- **Strict Type Safety**: TypeScript `strict: true`, `noUncheckedIndexedAccess: true`, `exactOptionalPropertyTypes: true`. Zero `any` or `@ts-ignore`.
- **Mobile-First Ergonomics**: Design and verify at `390x844` (mobile) first. Minimum `48x48px` touch targets, `env(safe-area-inset-*)` padding, and `tabular-nums` for numeric data.
- **Hard Drive Persistence**: Write all ADRs, domain specs, and technical spikes directly to `/docs/` — never leave specifications only in chat history.
- **Git & Authorship Hygiene**: Follow Conventional Commits (`feat(scope): ...`). **Never** add AI `Co-authored-by:` trailers unless explicitly requested by the repository owner.
- **360-Degree Verification**: Every Pull Request must pass the `standards-omni-role-review` checklist before merge.
```

### 2. Frontend Scoped `src/app/AGENTS.md` (or `src/frontend/AGENTS.md`) Template

```markdown
# Scoped Frontend & UI Engineering Rules (`src/app/AGENTS.md`)

## Active Skills & Helper References
When modifying files in this directory, load and enforce:
1. **Base Skills**: `role-frontend-developer`, `role-ui-ux-designer`, `design-engineering`, `role-performance-consultant`, `standards-code-style`
2. **Framework & Craft Helpers**:
   - Framework Guide: `role-frontend-developer/frameworks/<react|angular|vue|svelte>.md`
   - UI Craft Checklist: `design-engineering/references/ui-craft-and-ergonomics-checklist.md`
   - Code Style & Markup Rules: `standards-code-style/references/javascript-typescript-lint-rules.md`, `standards-code-style/references/html-css-scss-style-rules.md`
3. **Project Extension**: `/.agents/extensions/role-frontend-developer.md`, `/.agents/extensions/standards-code-style.md`
4. **Design System Docs**: `/docs/architecture/design-tokens.md`

## Local Directory Constraints
- **Component Purity & State**: Keep UI components declarative. Co-locate view state; lift shared state only through typed stores/signals or URL search params.
- **Zero "AI Slop"**: Use semantic design tokens extracted from Penpot (`--color-surface-*`, `--space-*`). Never hardcode arbitrary hex values or generic purple-blue gradients.
- **All 8 UI States**: Every data-driven view MUST explicitly implement: `Loading (Skeleton)`, `Empty`, `Partial`, `Ideal`, `Overflow / Long Text`, `Error (with Retry)`, `Offline`, and `Permission Denied`.
- **Compositor-Only Motion**: Animate exclusively `transform` and `opacity` (150ms–250ms) and honor `@media (prefers-reduced-motion: reduce)`.
- **Accessibility & Ergonomics**: Enforce WCAG 2.2 AA contrast, semantic HTML landmarks, keyboard focus rings (`:focus-visible`), `48x48px` tap targets, and `font-size >= 16px` on mobile inputs.
- **Boundary Rule**: Never import server-only database clients, filesystem modules, or private environment variables into client bundles.
```

### 3. Backend Scoped `src/server/AGENTS.md` (or `src/api/AGENTS.md`) Template

```markdown
# Scoped Backend & API Engineering Rules (`src/server/AGENTS.md`)

## Active Skills & Helper References
When modifying files in this directory, load and enforce:
1. **Base Skills**: `role-backend-developer`, `role-software-architect`, `role-security-consultant`, `role-legal-counsel`, `standards-code-style`
2. **Technical Helpers**:
   - Backend & API Guide: `role-backend-developer/references/` (and framework helper)
   - Code Style & SQL Rules: `standards-code-style/references/javascript-typescript-lint-rules.md`, `standards-code-style/references/sql-backend-and-test-style-rules.md`
   - Security & OWASP Audit: `role-security-consultant/references/`
   - GDPR & Privacy Engineering: `role-legal-counsel/references/gdpr-privacy-compliance-checklist.md`
3. **Project Extension**: `/.agents/extensions/role-backend-developer.md`, `/.agents/extensions/standards-code-style.md`
4. **Architecture & Domain Docs**: `/docs/architecture/api-contracts.md`, `/docs/domain/`

## Local Directory Constraints
- **Boundary Validation**: Validate 100% of inbound request bodies, query parameters, path params, and webhook payloads at the controller boundary using strict runtime schemas (e.g., Zod / Valibot).
- **Authentication & Authorization (BOLA Prevention)**: Verify both authentication session and resource-level ownership/tenant authorization on every mutation and sensitive query.
- **Idempotency & Concurrency**: Mutating endpoints (payments, bookings, status transitions, inventory updates) must support idempotency keys or atomic database transactions with optimistic/pessimistic locking.
- **Structured Error Envelope**: Return RFC 9457 Problem Details (or the project's standardized error JSON envelope) with machine-readable error codes. Never leak stack traces or raw SQL errors to clients.
- **PII & Log Scrubbing**: Never log raw passwords, session tokens, payment details, or unmasked PII (emails, national IDs, IP addresses) to stdout or error tracking services.
- **Database Discipline**: All schema changes require version-controlled migrations (`expand-and-contract` pattern for zero-downtime deploys). Avoid N+1 queries; enforce pagination limits on all collection endpoints.
```

### 4. Shared / Domain Scoped `src/shared/AGENTS.md` (or `packages/core/AGENTS.md`) Template

```markdown
# Scoped Shared Domain & Contracts Rules (`src/shared/AGENTS.md`)

## Active Skills & Helper References
When modifying files in this directory, load and enforce:
1. **Base Skills**: `role-software-architect`, `role-business-analyst`, `role-senior-code-reviewer`, `standards-code-style`
2. **Code Style Helpers**: `standards-code-style/references/formatting-naming-and-structure.md`, `standards-code-style/references/javascript-typescript-lint-rules.md`
3. **Project Extension**: `/.agents/extensions/role-software-architect.md`, `/.agents/extensions/standards-code-style.md`
4. **Domain Model Docs**: `/docs/domain/`

## Local Directory Constraints
- **Zero Framework or Runtime I/O Coupling**: Code in `src/shared/` must be 100% pure TypeScript/JavaScript. Do NOT import DOM APIs (`window`, `document`), UI frameworks (React, Angular, Vue), Node.js built-ins (`fs`, `net`, `crypto`), or ORM/database drivers.
- **Single Source of Truth Contracts**: Define shared DTOs, validation schemas (Zod), domain value objects, branded types (e.g., `UserId`, `OrganizationId`, `CurrencyAmount`), and pure business calculation functions here.
- **Ubiquitous Language**: Type names, enum values, and domain error codes must match the exact terminology defined in `/docs/domain/glossary.md`.
- **100% Unit Testability**: Every domain calculation and validator in `src/shared/` must be deterministically testable in isolation without network or database mocks.
```

### 5. QA & Testing Scoped `tests/AGENTS.md` (or `e2e/AGENTS.md`) Template

```markdown
# Scoped QA & Test Automation Rules (`tests/AGENTS.md`)

## Active Skills & Helper References
When modifying files in this directory, load and enforce:
1. **Base Skills**: `role-qa-engineer`, `role-security-consultant`, `role-performance-consultant`, `standards-code-style`
2. **Technical Helper Checklist**:
   - `role-qa-engineer/references/testing-pyramid-playwright-checklist.md`
   - `standards-code-style/references/sql-backend-and-test-style-rules.md`
3. **Project Extension**: `/.agents/extensions/role-qa-engineer.md`, `/.agents/extensions/standards-code-style.md`

## Local Directory Constraints
- **User-Facing Locators Only**: Prioritize `getByRole`, `getByLabel`, and `getByText` over `getByTestId`. Strictly forbid brittle CSS class or XPath selectors.
- **Zero Flakiness Policy**: Never use arbitrary sleeps (`page.waitForTimeout()`). Use web-first auto-retrying assertions (`await expect(locator).toBeVisible()`) and deterministic network/clock control.
- **Test Isolation**: Each test must be completely independent and parallel-safe (`fullyParallel: true`), seeding its own isolated state or rolling back database transactions.
- **Mobile & Accessibility Gates**: E2E suites must run against mobile viewports (`390x844` iPhone / `412x915` Pixel with touch enabled) and execute automated `@axe-core/playwright` WCAG 2.2 AA scans on every critical route.
```

---

## Scaffolding Checklist for Agents

- [ ] **Stack Discovery**: Inspected project manifest and `/docs/adr/` to identify exact frontend, backend, database, and testing frameworks before generating scoped `AGENTS.md` files.
- [ ] **Directory Coverage**: Created root `AGENTS.md` plus scoped `AGENTS.md` files for frontend, backend, shared domain, and test directories.
- [ ] **Extension Skeleton**: Created `<project-root>/.agents/extensions/` with starter delta files for core project skills.
- [ ] **Helper Linking**: Linked every scoped `AGENTS.md` to its corresponding base skill and `frameworks/` or `references/` helper checklist.
- [ ] **Formatting Verification**: Verified zero corrupted box-drawing characters, 100% English prose, and valid Markdown syntax across all generated files.

---

## Continuous Evolution & Self-Improvement Mandate
- **Baseline Floor, Not a Ceiling**: The templates above provide the foundational scaffolding baseline. As a project grows (e.g., adding mobile Capacitor wrappers, background worker queues, or infrastructure-as-code directories), proactively scaffold new scoped `AGENTS.md` files for those workspaces.
- **Knowledge Feedback Loop**: When agents repeatedly encounter ambiguity in a directory, codify the clarification into that directory's `AGENTS.md` or `.agents/extensions/<skill-name>.md`. If the pattern is universally beneficial across all projects, upstream it into this base skill.
