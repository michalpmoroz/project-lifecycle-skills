# AGENTS.md - Directives for AI Coding Assistants

> [!IMPORTANT]
> This repository contains **Project Lifecycle Skills**, a universal, framework-agnostic suite of skills, standards, and roles designed for AI-driven software development across all lifecycle phases.
> Any AI agent reading, executing, maintaining, or extending these skills **must strictly comply** with the rules set forth in this document.

---

## 1. Non-Negotiable Governance Rules

All skills in this repository are governed by the standards codified in [`standards-skills-governance`](file:///standards-skills-governance/SKILL.md). Every AI agent must internalize and enforce these rules:

### Rule A: Project-Agnostic Mandate
- Skills describe **how** to execute processes, apply techniques, and maintain standards. They must **never** describe a specific project being built.
- Prohibited in all base skills: hardcoding single-project assumptions, single-project URLs, bespoke database schemas, or single-domain business logic.
- **Multi-Domain Examples Policy**: Illustrative domain terms (e.g., civic/elections, healthcare, e-commerce, real estate, SaaS) are permitted **only as examples** and must always be accompanied by parallel multi-domain alternatives.

### Rule B: Technology Best Practices as Helper Files
- Framework-specific and library-specific implementation guides must live in `frameworks/` or `references/` subdirectories alongside `SKILL.md`, **not** directly inside `SKILL.md`.
- `SKILL.md` defines the role, core principles, workflows, and references to these helper files.

### Rule C: Project Extension Protocol (Zero Modification of Base Skills)
- Target projects consume skills without modifying the base skill repository.
- Projects create project-specific extensions under `<project-root>/.agents/extensions/<skill-name>.md`.
- When an agent activates a skill, it must check for `<project-root>/.agents/extensions/<skill-name>.md` and merge project overrides and additions on top of the base skill defaults.
- Extensions contain only deltas (overrides and additions), never duplicate base skill text.

### Rule D: Documentation Cross-Linking
- When activated, skills read project documentation under `<project-root>/docs/` for context.
- Target project docs declare applicable skills in their YAML frontmatter:
  ```yaml
  ---
  skills: [role-archivist, role-backend-developer, role-legal-counsel]
  ---
  ```
- Skills scan for docs that tag them in frontmatter. Bidirectional traceability between documentation and skills must be maintained.

### Rule E: English-Only in Base Skills
- All prose, code comments, markdown files, labels, and documentation in this repository must be 100% English.
- Localized/native-language strings belong exclusively in project workspaces (e.g. user-facing UI text, i18n dictionaries, custom URL slugs defined in project extensions).

### Rule F: AI Agent Authorship & Attribution Policy
- **AI agents must NEVER inject themselves as commit authors or co-authors (`Co-authored-by:`)** in git commits, pull requests, or release metadata unless explicitly instructed by the user.
- Commits must use the user's configured identity (`git config user.name` / `git config user.email`).
- Do not append AI attribution signatures, bot emails, or assistant branding to commit messages.

---

## 2. Hard Drive Persistence Mandate

- **No ephemeral storage**: Any architectural specification, technical spike, ADR, competitor benchmark, threat model, or business analysis generated during lifecycle execution **must be written directly to the hard drive** in `<project-root>/docs/`.
- Do not leave critical plans solely in assistant scratchpads or temporary chat contexts.
- Living documentation must be versioned in Git alongside the project codebase.

---

## 3. Sequential Lifecycle Progression

When orchestrating a project, AI agents must adhere to the phased progression defined in [`project-lifecycle`](file:///project-lifecycle/SKILL.md) and [`init-project`](file:///init-project/SKILL.md):

```
Phase 1: Discovery & Tech Research
Phase 2: Architecture & ADRs
Phase 3: Governance & Issue Tracking
Phase 4: Living Documentation Setup (/docs/)
Phase 5: Visual Design & Design Tokens (Penpot / Figma)
Phase 6: Core Scaffolding & CI/CD
Phase 7: Design System Code Implementation
Phase 8: Feature Delivery & Verification Loops
```

**Agents must not jump ahead**:
- Do not write application feature code before visual design tokens and system architecture are locked.
- Do not finalize a product backlog before completing the universal [`standards-web-essentials`](file:///standards-web-essentials/SKILL.md) audit and the [`standards-omni-role-review`](file:///standards-omni-role-review/SKILL.md) 360-degree review.

---

## 4. Omni-Role 360-Degree Review Protocol

Every deliverable, feature specification, task refinement, and Pull Request must pass the Multi-Perspective 360-Degree Review ([`standards-omni-role-review`](file:///standards-omni-role-review/SKILL.md)). Agents must evaluate:

1. **Product Owner**: User value and acceptance criteria.
2. **Business Analyst**: Edge cases and domain consistency.
3. **Software Architect**: Scalability, modular boundaries, and ADR compliance.
4. **UI/UX Designer & Design Engineer**: Mobile ergonomics (44px touch targets), micro-interactions, layout polish.
5. **Frontend & Mobile Developer**: Signals/state cleanliness, bundle size, offline readiness.
6. **Backend Developer**: Idempotency, schema validation, API contracts.
7. **QA Engineer**: Test pyramid coverage (Unit, Integration, Playwright E2E).
8. **Security Consultant**: OWASP Top 10, sanitization, secret management.
9. **Performance Consultant**: Core Web Vitals (LCP, INP, CLS), mobile 90+ Lighthouse targets.
10. **Legal Counsel**: GDPR compliance, privacy, cookies, liability shielding.
11. **DevOps Engineer**: GitHub Actions CI/CD reproducibility.
12. **Archivist**: Documentation freshness in `/docs/` and frontmatter skill tagging.
13. **SEO/GEO Specialist**: Sitemaps, Open Graph metadata, JSON-LD structured data.
14. **Senior Code Reviewer**: Type safety, lint rules, clean code principles.

---

## 5. Free & Open-Source First Policy

Whenever recommending, selecting, or provisioning tools, libraries, infrastructure, or services, agents must strictly follow [`tooling-free-and-open-source`](file:///tooling-free-and-open-source/SKILL.md):
- Always pair or prioritize 100% free or open-source software (FOSS) and generous perpetual free tiers (e.g., Penpot, Supabase/PocketBase, Playwright, Cloudflare Pages/Workers, Docker, GitHub Actions free tiers).
- Avoid proprietary paid-tier dependencies unless requested by the user.

---

## 6. Repository Maintenance Checklist for Agents

When editing or submitting changes to this repository:
- [ ] Confirm no single-project logic is introduced into base skills.
- [ ] Ensure all helper scripts or framework-specific guides are placed under `<skill>/frameworks/` or `<skill>/references/`.
- [ ] Ensure all new files are in standard English.
- [ ] Verify that Git commits follow Conventional Commits format (`feat:`, `fix:`, `docs:`, `chore:`).
- [ ] Ensure **zero** AI co-authorship tags (`Co-authored-by:`) are inserted into commit messages.
