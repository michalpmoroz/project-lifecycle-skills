# Project Lifecycle Skills Changelog

All notable changes to the shared **Project Lifecycle Skills** (`project-lifecycle-skills`) are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

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
