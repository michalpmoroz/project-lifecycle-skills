---
name: standards-skills-governance
description: >-
  Enforces the core architectural and authorship governance rules governing how Project Lifecycle Skills are authored, extended, and consumed: project-agnosticism, technology helper files, project extension protocol, documentation cross-linking, English-only content, AI authorship attribution, and tool version recording with continuous knowledge evolution.
---

# Skills Governance Standards (`standards-skills-governance`)

## Mission
Preserve the integrity, reusability, and longevity of the shared **Project Lifecycle Skills** pack by enforcing seven non-negotiable hard rules. These rules must be applied by any agent or contributor who authors, edits, consumes, or extends any skill file.

> [!IMPORTANT]
> These rules are **non-negotiable**. Any violation found during a review must be flagged as a blocking issue and corrected before merging or archiving.

---

## Rule A — Project-Agnostic Mandate & Multi-Domain Examples

**All base skill files must be 100% project-agnostic.**

- Skills describe **how** to execute processes, apply architectural patterns, and enforce quality standards — never **what** single bespoke project is being built.
- **Strictly Prohibited in Base Skills**: Hardcoding single-project assumptions, project codenames, single-project live URLs, bespoke single-application database schemas, or single-domain business rules as universal requirements.
- **Multi-Domain Examples Policy**: Illustrative domain terms are permitted **only as concrete examples** and must always pair multiple diverse domains alongside each other (e.g., E-commerce, Healthcare, FinTech, Civic/GovTech, B2B SaaS, Real Estate) to preserve universality.
- Content specific to one active project belongs exclusively in the target project's extension files (`.agents/extensions/<skill-name>.md`, see Rule C) or in `<workspace>/docs/` (see Rule D).

### Illustrative Examples Across Multiple Domains
```markdown
# Search & Geotargeting Examples:
- Civic / GovTech: "Polling stations or representatives near [postal code]"
- Healthcare: "Urgent care clinics accepting [insurance] near [city]"
- E-commerce / Retail: "In-stock fulfillment pickup lockers near [neighborhood]"
- Real Estate: "Commercial properties for lease within [district]"

# XML Sitemap Partition Examples:
- E-commerce: `sitemap-products.xml`, `sitemap-categories.xml`, `sitemap-brands.xml`
- Healthcare: `sitemap-practitioners.xml`, `sitemap-clinics.xml`, `sitemap-specialties.xml`
- Civic / Directory: `sitemap-profiles.xml`, `sitemap-regions.xml`, `sitemap-organizations.xml`
- B2B SaaS: `sitemap-integrations.xml`, `sitemap-templates.xml`, `sitemap-docs.xml`
```

---

## Rule B — Technology Best Practices as Helper Files

**Technology-specific and deep-dive domain checklists live in named helper files within the skill's subdirectory (`frameworks/` or `references/`), not bloated directly inside `SKILL.md`.**

- Each skill directory may contain `frameworks/` or `references/` subdirectories:
  ```text
  role-frontend-developer/
    SKILL.md                                       <- Role mission, core principles, activation & helper index
    frameworks/
      react.md                                     <- React 19.x / Next.js 15.x implementation rules
      angular.md                                   <- Angular 19.x / 18.x Signals & Standalone rules
      vue.md                                       <- Vue 3.5+ / Nuxt 3.14+ rules
  role-qa-engineer/
    SKILL.md                                       <- QA role definition & workflow
    references/
      testing-pyramid-playwright-checklist.md      <- Playwright 1.49+, Vitest 2.x, axe-core 4.10+ checklist
  ```
- `SKILL.md` defines the Senior/Staff role mission, core architectural principles, activation protocol, and an explicit directory table stating **when to load each helper file**.
- This keeps `SKILL.md` scannable and token-efficient while allowing deep, actionable checklists in helper files to evolve independently.

---

## Rule C — Project Extension Protocol (Zero Modification of Base Skills)

**Target projects consume and customize skills without modifying base skill files.**

- The base skill repository must never be edited to hardcode a single project's bespoke conventions.
- Target projects create delta extension files in their own workspace under:
  ```text
  <workspace>/
    .agents/
      extensions/
        role-frontend-developer.md          <- Project-specific UI stack & component overrides
        role-legal-counsel.md               <- Project-specific regulatory statutes & jurisdictions
        role-seo-geo-specialist.md          <- Project-specific sitemap partitions & Schema.org entities
        standards-git-and-releases.md       <- Project-specific commit scopes & release branches
  ```
- **Activation Protocol**: Whenever an agent activates any skill, it **must** check whether `<workspace>/.agents/extensions/<skill-name>.md` exists. If present, its overrides and additions take precedence over base skill defaults.
- Extension files must contain **only delta rules** (Project Context, Overrides, Additions) and never copy-paste base skill content.

### Extension File Template
```markdown
# <Skill Name> — Project Extension

## Project Context
<!-- Brief summary of the project domain, target users, and regulatory/technical environment. -->

## Overrides
<!-- Rules that replace specific base skill defaults for this repository (e.g., custom breakpoints, custom URL slugs, specific ORM choice). -->

## Additions
<!-- Domain-specific invariants, bespoke checklists, or third-party integrations unique to this project. -->
```

---

## Rule D — Documentation Cross-Linking & Hard Drive Persistence

**Skills read `<workspace>/docs/` for context; project documentation declares governing skills in YAML frontmatter.**

- **Hard Drive Persistence Mandate**: No architectural specification, ADR, domain model, threat model, spike benchmark, or audit report may remain solely in ephemeral chat context or scratchpads. All project knowledge must be persisted to `<workspace>/docs/`.
- Project documentation files in `<workspace>/docs/` must declare which skills govern or consume them via YAML frontmatter:
  ```yaml
  ---
  title: "Authentication & Session Architecture"
  status: "Accepted"
  updated: "2026-11-02"
  skills: [role-software-architect, role-backend-developer, role-security-consultant]
  ---
  ```
- When activated, skills scan `<workspace>/docs/` for documents listing their skill name in `skills:` frontmatter (and related domain folders such as `/docs/adr/`, `/docs/domain/`, `/docs/architecture/`) to align implementation with established decisions.
- `role-archivist` enforces bidirectional traceability across all `/docs/` files on every Pull Request.

---

## Rule E — 100% English-Only in Base Skills

**All content in the base skill repository must be written in clean, standard English.**

- Every `.md` file, code snippet, comment, frontmatter description, and checklist in the base skill pack must be 100% English.
- **Formatting Hygiene**: Files must never contain corrupted encoding artifacts (such as box-drawing characters `├──` replacing standard spaces) or broken markdown tables.
- **Project-Level Localization**: Native-language strings belong exclusively in the target project workspace (e.g., `i18n/` dictionaries, user-facing copy, and localized URL slugs defined in `.agents/extensions/`).

---

## Rule F — AI Agent Authorship & Attribution Policy

**AI agents must NEVER inject themselves as commit authors or co-authors (`Co-authored-by:`) unless explicitly instructed by the user.**

- **Default Behavior**: All git commits, pull requests, and release tags must use the developer's configured Git identity (`git config user.name` and `git config user.email`) with zero AI attribution trailers, bot emails, or `"Generated by..."` footers.
- **One-Time Project Inception Option**: When initializing a brand-new repository (`init-project`), the agent may ask the user once whether they want AI co-authorship attribution enabled. If declined or unprompted, never include AI attribution.

---

## Rule G — Tool Version Recording & Continuous Knowledge Evolution

**All helper files and checklists must record their baseline tool/framework versions, and agents must treat checklists as a living floor — never a static ceiling.**

1. **Explicit Tooling & Standard Version Baseline**:
   - Every helper file in `references/` or `frameworks/` (and technical checklists in `SKILL.md`) must explicitly declare the baseline versions of frameworks, libraries, and standards used to establish its rules (e.g., `Playwright 1.49+`, `Vitest 2.x`, `WCAG 2.2 AA`, `OWASP Top 10:2021`, `Schema.org v28+`, `GDPR EU 2016/679`, `Tailwind CSS 4.0`).
   - When a target project uses a newer or older major version (verified via `package.json`, lockfiles, or ADRs), the agent must account for version deltas and record project-specific version constraints in `.agents/extensions/<skill-name>.md`.

2. **Checklists Are a Baseline Floor, Not a Ceiling**:
   - Pre-defined checklists represent the minimum Senior/Staff engineering bar. Agents must **not** mechanically stop thinking once boxes are ticked; they must proactively analyze architectural edge cases, runtime failure modes, and domain-specific risks beyond the checklist.

3. **Continuous Knowledge Capture Loop**:
   - **Project-Specific Learnings**: When a novel bug, domain edge case, or framework quirk is discovered during implementation or code review, capture it immediately in `<workspace>/.agents/extensions/<skill-name>.md` or `<workspace>/docs/adr/` so the mistake is never repeated in that project.
   - **Universal Reusable Patterns**: When a newly discovered pattern, anti-pattern, or tool upgrade is 100% project-agnostic and universally valuable across industries, propose or codify it into the appropriate base skill helper file (`references/` or `frameworks/`) with updated version metadata.

---

## Governance Audit Checklist (Verify on Every Skill Edit)

- [ ] **Rule A (Project-Agnostic)**: Zero single-project assumptions or hardcoded bespoke schemas; concrete examples span multiple industries (E-commerce, Healthcare, FinTech, Civic, SaaS).
- [ ] **Rule B (Helper Modularity)**: Deep framework/tool guides and exhaustive checklists are organized in `frameworks/` or `references/` and indexed in `SKILL.md`.
- [ ] **Rule C (Extension Protocol)**: Base skill includes activation instructions checking `.agents/extensions/<skill-name>.md`.
- [ ] **Rule D (Docs Cross-Linking)**: Base skill instructs checking `<workspace>/docs/` frontmatter (`skills: [...]`) and persisting artifacts to disk.
- [ ] **Rule E (English & Formatting)**: 100% English prose and comments; zero corrupted whitespace characters or broken Markdown.
- [ ] **Rule F (Zero Unsolicited AI Attribution)**: No `Co-authored-by:` bot trailers in commit guidelines or templates.
- [ ] **Rule G (Version Baseline & Evolution)**: Exact tool/standard versions are recorded at the top of helper checklists, and the Continuous Evolution Mandate is included.
