---
name: standards-skills-governance
description: >-
  Enforces the core architectural and authorship governance rules governing how Project Lifecycle Skills are authored, extended, and consumed: project-agnosticism, technology helper files, project extension protocol, documentation cross-linking, and English-only content.
---

# Skills Governance Standards (`standards-skills-governance`)

## Mission
Preserve the integrity, reusability, and longevity of the shared **Project Lifecycle Skills** pack by enforcing five non-negotiable hard rules. These rules must be applied by any agent or contributor who authors, edits, or extends any skill file.

> [!IMPORTANT]
> These rules are **non-negotiable**. Any violation found during a review must be flagged as a blocking issue and corrected before merging or archiving.

---

## Rule A  -  Project-Agnostic Mandate & Multi-Domain Examples

**All skill files must be project-agnostic.**

- Skills describe **how** to do something (process, technique, standard), never **what** specific single project is being built.
- Prohibited in any skill file: hardcoding single-project assumptions, hardcoding dedicated live URLs, or coupling logic to one bespoke platform.
- **Domain Examples Policy**: Illustrative domain terms (e.g. elections, healthcare, e-commerce, real estate, finance) are allowed **only as examples**. Whenever domain-specific terminology is used in an example, provide multi-domain alternatives alongside it to preserve universality.
- If comprehensive content is specific to one active implementation, it belongs exclusively in the project's extension files (see Rule C) or in `/docs/`.

### Illustrative Examples Across Multiple Domains
```md
# Search / Geotargeting Examples:
- Civic / Elections: "Representatives near [postal code]"
- Healthcare: "Clinics near [city]"
- Real Estate: "Properties near [neighborhood]"

# XML Sitemap Partition Examples:
- Civic: `sitemap-profiles.xml`, `sitemap-districts.xml`
- Medical: `sitemap-doctors.xml`, `sitemap-clinics.xml`
- E-commerce: `sitemap-products.xml`, `sitemap-categories.xml`
```

---
## Rule B  -  Technology Best Practices as Helper Files

**Technology-specific knowledge lives in named helper files within the skill's subdirectory, not in `SKILL.md` itself.**

- Each skill may contain a `frameworks/` or `references/` subdirectory.
- Each framework or tool gets its own named `.md` file:
  ```
  role-frontend-developer/
    SKILL.md                  <- role definition (framework-agnostic)
    frameworks/
      react.md                <- React/Next.js best practices
      angular.md              <- Angular best practices
      vue.md                  <- Vue/Nuxt best practices
  ```
- `SKILL.md` states which helper files exist and when to load them.
- Helper files may be framework-specific, library-specific, or tool-specific (e.g. `role-backend-developer/references/payload-cms.md`).
- This ensures `SKILL.md` stays readable and stable while technology details evolve independently.

---

## Rule C  -  Project Extension Without Modification

**Projects extend skills without modifying the base skill files.**

- The base skill pack (installed at the agent's config location) must never be edited to accommodate a single project's needs.
- Projects create extension files in their workspace under:
  ```
  <workspace>/
    .agents/
      extensions/
        role-frontend-developer.md    <- project-specific overrides & additions
        role-legal-counsel.md         <- project-specific legal domain rules
        role-seo-geo-specialist.md    <- project-specific sitemaps, GEO hierarchy
        standards-git-and-releases.md <- project-specific commit scopes
  ```
- **Naming convention**: `<skill-name>.md` inside `.agents/extensions/`.
- When a skill is activated, it **must** check for a corresponding extension file in `.agents/extensions/` and merge its rules on top of the base skill defaults.
- Extension files may:
  - Override defaults (e.g. URL slug conventions, code style rules).
  - Add new rules for tools not yet defined in the base skill.
  - Suppress or replace base examples with project-relevant ones.
- Extension files must **not** duplicate base skill content  -  only delta rules.

### Extension File Template
```md
# [Skill Name]  -  Project Extension

## Project Context
<!-- Brief description of the project domain. -->

## Overrides
<!-- Rules that replace base skill defaults for this project. -->

## Additions
<!-- New rules, tool configs, or conventions not in the base skill. -->
```

---

## Rule D  -  Documentation Cross-Linking

**Skills read project documentation for context; documentation marks which skills it involves.**

- Skills, when activated, should check for relevant `.md` files in `<workspace>/docs/` to gain project context.
- Project documentation files that describe processes involving specific skills should declare which skills they are relevant to using a YAML frontmatter tag:
  ```yaml
  ---
  skills: [role-archivist, role-backend-developer, role-legal-counsel]
  ---
  ```
- Skills should scan for docs bearing their own name in the `skills` frontmatter list and use that content to inform their decisions.
- This enables bidirectional traceability: skills know which docs apply to them; docs declare which skills are responsible for implementing or auditing them.
- The `role-archivist` skill is responsible for maintaining this cross-linking discipline across all `/docs/` files.

---

## Rule E  -  English-Only in Skills

**All content in the skill pack must be written in English.**

- Every `.md` file in the base skill pack must be 100% English: instructions, examples, comments, frontmatter, placeholder text, and URLs.
- **Project-level files** (in the project workspace) may contain native-language content **only** for user-facing text:
  - UI copy, labels, and error messages targeted at end users.
  - URL slugs that the user wants in their native language (e.g. `/polityka-prywatnosci` instead of `/privacy-policy`).
  - i18n translation files.
- When a skill provides a default English example (e.g. `/privacy-policy`), the project extension file may override it with a localized version. The base skill always uses the English default.

### Enforcement
Any agent reviewing a skill file or extension file must verify:
- [ ] All prose, headings, and code comments are in English.
- [ ] No project-specific words in languages other than English appear in base skill files.
- [ ] Native language content in project files is limited to user-facing strings only.

---

## Rule F  -  AI Agent Authorship & Attribution Policy

**AI Agents must never add themselves as commit authors or co-authors (`Co-authored-by:`) unless the user explicitly requests it.**

- **Default Behavior**: AI agents commit strictly under the developer/user's configured identity without injecting agent names, bot emails, or tool attribution tags.
- **One-Time Project Inception Protocol**: When initializing a brand-new repository (e.g. creating the first commit during project setup), the agent is permitted to ask the user once whether they prefer AI co-authorship attribution in commits.
- If the user declines, does not answer, or in all subsequent operations, no AI co-authorship metadata may be included.

---

## Governance Checklist (Apply on Every Skill Edit)

Before committing or archiving any change to the skill pack, verify:

- [ ] **Rule A**: Base skills remain project-agnostic; any domain-specific terms are strictly illustrative and balanced across multiple industries.
- [ ] **Rule B**: Technology-specific details are in `frameworks/` or `references/` helper files, not in `SKILL.md`.
- [ ] **Rule C**: No project customization is embedded in the base skill  -  project extension files are used instead.
- [ ] **Rule D**: Any new skill references which project docs it expects; any new project doc declares relevant skills in frontmatter.
- [ ] **Rule E**: All content is in English; native language restricted to project-level user-facing strings.
- [ ] **Rule F**: AI agents never add themselves as co-authors unless explicitly requested by the user.
