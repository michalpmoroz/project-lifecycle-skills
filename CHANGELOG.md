# Project Lifecycle Skills Changelog

All notable changes to the shared **Project Lifecycle Skills** (`project-lifecycle-skills`) are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [v1.1.0] - 2026-11-02 13:45:00 CET
### Added
- **Rule G (Tool Version Recording & Continuous Knowledge Evolution)** in `standards-skills-governance` and `AGENTS.md`:
  - Mandates explicit recording of baseline tool, framework, and standard versions in every helper file.
  - Establishes checklists as a minimum quality floor rather than a ceiling, instructing agents to continuously expand knowledge (persisting project-specific rules to `<workspace>/.agents/extensions/` and universal patterns to `project-lifecycle-skills`).
- **`standards-code-style` Skill & 4 Tool-Agnostic Baseline Style/Lint References**:
  - `standards-code-style/SKILL.md`: Universal code formatting, naming, structure, complexity, and static analysis rules ready to translate into project linters/formatters (Prettier, Biome, ESLint 9, Stylelint, Ruff, SQLFluff) without storing tool-specific config files in the base skill pack.
  - `standards-code-style/references/formatting-naming-and-structure.md`
  - `standards-code-style/references/javascript-typescript-lint-rules.md`
  - `standards-code-style/references/html-css-scss-style-rules.md`
  - `standards-code-style/references/sql-backend-and-test-style-rules.md`
- **32 Granular, Single-Concern Framework & Reference Checklists** (preserving context window by loading only the exact technology or layer slice needed for a task):
  - **Performance Consultant (10 modular files)**:
    - `role-performance-consultant/references/frontend-html-and-assets-checklist.md`
    - `role-performance-consultant/references/frontend-css-scss-less-checklist.md`
    - `role-performance-consultant/references/frontend-js-ts-runtime-checklist.md`
    - `role-performance-consultant/references/frontend-build-bundling-and-delivery.md`
    - `role-performance-consultant/frameworks/angular-performance.md`
    - `role-performance-consultant/frameworks/react-nextjs-performance.md`
    - `role-performance-consultant/frameworks/vue-nuxt-performance.md`
    - `role-performance-consultant/frameworks/svelte-performance.md`
    - `role-performance-consultant/references/backend-api-and-database-performance.md`
    - `role-performance-consultant/references/caching-network-and-ci-performance.md`
  - **Frontend & Mobile Engineering (9 modular files)**:
    - `role-frontend-developer/frameworks/angular.md`
    - `role-frontend-developer/frameworks/react.md`
    - `role-frontend-developer/frameworks/vue.md`
    - `role-frontend-developer/frameworks/svelte.md`
    - `role-frontend-developer/references/html-and-accessibility-checklist.md`
    - `role-frontend-developer/references/css-scss-tailwind-checklist.md`
    - `role-frontend-developer/references/typescript-and-i18n-checklist.md`
    - `role-mobile-developer/references/pwa-service-worker-and-offline-sync.md`
    - `role-mobile-developer/references/capacitor-native-and-store-compliance.md`
  - **Backend, Architecture, DevOps & Security (9 modular files)**:
    - `role-backend-developer/references/api-design-and-contracts-checklist.md`
    - `role-backend-developer/references/database-schema-migrations-and-rls.md`
    - `role-backend-developer/references/auth-authorization-and-resilience.md`
    - `role-backend-developer/frameworks/node-typescript-backend.md`
    - `role-backend-developer/frameworks/payload-supabase-pocketbase.md`
    - `role-software-architect/references/architecture-patterns-checklist.md`
    - `role-devops-engineer/references/cicd-containers-infra-checklist.md`
    - `role-security-consultant/references/owasp-web-api-and-access-checklist.md`
    - `role-security-consultant/references/crypto-supply-chain-and-ai-security.md`
  - **QA, Code Review, Design Engineering, SEO/GEO & Legal (5 modular files)**:
    - `role-qa-engineer/references/testing-pyramid-playwright-checklist.md`
    - `role-senior-code-reviewer/references/pr-code-review-checklist.md`
    - `design-engineering/references/ui-craft-and-ergonomics-checklist.md`
    - `role-seo-geo-specialist/references/technical-seo-geo-schema-checklist.md`
    - `role-legal-counsel/references/gdpr-privacy-compliance-checklist.md`
### Changed
- **Upgraded All 31 Skills to Principal / Staff Engineering Depth**:
  - Added Activation, Project Extension (`.agents/extensions/<skill-name>.md`), and Documentation Cross-Linking (`/docs/` frontmatter) protocols across all role and standard skills.
  - Added multi-domain examples (E-commerce, Healthcare, FinTech, Civic/GovTech, SaaS) and explicit verification checklists across all roles.
### Fixed
- **`project-agent-scaffolder/SKILL.md`**: Removed corrupted box-drawing characters (`├──`) and added complete 3-tier resolution instructions and scoped `AGENTS.md` templates.

## [v1.0.0] - 2026-11-01 21:30:00 CET
### Added
- **Official Public Release (v1.0.0)** of the universal **Project Lifecycle Skills** suite.
- **AGENTS.md**: Dedicated hard notes and behavioral directives for AI Coding Assistants (enforcing Governance Rules A-F, hard drive persistence, and zero AI co-authorship).
- **README.md**: Comprehensive documentation detailing the 30-skill capability matrix, recommended repository structure, installation modes, project extension mechanics, and frontmatter documentation cross-linking.
- **Repository Setup**: Initialized standalone repository with MIT license, clean Git history, and public GitHub distribution.

## [v0.11.0] - 2026-10-30 02:30:00 CET
### Added
- **`standards-skills-governance`**: New meta-skill codifying five non-negotiable architectural governance rules:
  - **Rule A**  -  Project-Agnostic Mandate: Skills must never contain project-specific keywords, entity names, or business logic.
  - **Rule B**  -  Technology Helper Files: Technology-specific best practices live in `frameworks/` or `references/` subdirectory files alongside `SKILL.md`.
  - **Rule C**  -  Project Extension Protocol: Projects extend skills by placing override files in `.agents/extensions/<skill-name>.md` without modifying base skills.
  - **Rule D**  -  Documentation Cross-Linking: Skills read relevant project `/docs/` files; docs declare applicable skills in YAML frontmatter (`skills: [...]`).
  - **Rule F (AI Authorship & Attribution Policy)** added to standards-skills-governance and standards-git-and-releases: AI agents must never inject themselves as commit authors or co-authors (Co-authored-by:) unless the user explicitly requests it (default: disabled; one-time setup question allowed on new repositories).
- **Loosened Rule A (Multi-Domain Examples)**: Allowed illustrative domain terminology (e.g. civic/elections, healthcare, real estate, e-commerce) when accompanied by multi-domain alternative examples to maintain framework universality.
  - **Rule E**  -  English-Only: All base skill content must be in English; native language allowed only in project-level user-facing strings.
### Fixed
- **Project-Agnosticism Cleanup** across all skills:
  - `role-legal-counsel`: Removed domain-specific legal domain references; generalized to cover any platform processing public figures' data.
  - `role-seo-geo-specialist`: Replaced domain-specific sitemaps and GEO examples with generic equivalents; moved domain specifics to project extension responsibility.
  - init-project: Replaced domain-specific wording in Steps 1 and 6 with generic equivalents.
  - `standards-omni-role-review`: Replaced domain-specific audience and edge-case examples with generic equivalents.
  - standards-git-and-releases: Removed domain-specific commit scopes; replaced with generic functional scopes.
  - `role-github-admin/references/github-connection-guide.md`: Replaced project-codename example with generic `my-app` placeholder.
  - `CHANGELOG.md`: Removed project directory path references and domain-specific feature descriptions from historical entries.

## [v0.10.0] - 2026-10-30 00:10:00 CET
### Fixed
- **YAML Frontmatter Repairs**:
  - standards-git-and-releases: Removed domain-specific commit scopes; replaced with generic functional scopes.
- **Language Neutrality & Internationalization in Skills**:
  - Cleaned all project-specific Polish URLs and references from universal skills.
  - Standardized boilerplate routes to clean English slugs (`/privacy`, `/terms`, `/accessibility`, `/contact`, `/changelog`).
- **Package Identity & Version Normalization**:
  - Renamed package to `project-lifecycle-skills` and normalized pre-1.0 development versions to `0.x.y`.

## [v0.9.0] - 2026-10-23 19:45:00 CEST
### Added
- **`role-seo-geo-specialist`**: Dedicated skill for search engine crawlability, geographic discovery (GEO targeting), semantic JSON-LD structured data, Open Graph/Twitter dynamic share cards, XML sitemaps, and robots.txt.
- **`standards-omni-role-review`**: Mandatory 360-degree review protocol requiring every issue, feature specification, and Pull Request to be explicitly reviewed against all specialized perspectives.
- **`standards-web-essentials`**: Comprehensive catalogue of universal boilerplate web features that must be evaluated and elicited prior to backlog finalization.

## [v0.8.0] - 2026-10-23 18:35:00 CEST
### Added
- **`role-legal-counsel`**: Comprehensive skill covering GDPR compliance, defamation defense, Terms of Service, Privacy Policies, Cookie compliance, and editorial liability shielding.

## [v0.7.0] - 2026-10-23 16:20:00 CEST
### Added
- **Hard Drive Repository Persistence Mandate**: Required all research, specifications, blueprints, and competitor analyses to be persisted directly to `<workspace>/docs/` on the physical hard drive.

## [v0.6.0] - 2026-10-23 15:50:00 CEST
### Changed
- **Separation of Visual Design vs. Code Implementation**:
  - Formalized sequence: Visual Design in Penpot -> Technical Scaffolding -> Design System Code -> Feature Delivery.

## [v0.1.0] - 2026-10-23 02:30:00 CEST
### Initial Baseline
- Foundational skills covering end-to-end software delivery.
