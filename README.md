# 🧭 Project Lifecycle Skills

[![Version](https://img.shields.io/badge/version-1.1.0-blue.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Standards: Project--Agnostic](https://img.shields.io/badge/Standards-Project--Agnostic-success.svg)](standards-skills-governance/SKILL.md)
[![AI Attribution Policy](https://img.shields.io/badge/AI%20Attribution-Zero%20Co--Author-red.svg)](AGENTS.md)

**Project Lifecycle Skills** is an end-to-end, project-agnostic suite of operational skills, Principal/Staff-grade engineering roles, framework-specific guides, and exhaustive verification checklists for AI coding agents and human engineering teams.

Designed to eliminate "AI coding hallucinations", superficial reviews, and context-window bloat, this skillset enforces phased project lifecycles, rigorous 360-degree omni-role reviews, architectural governance, mobile-first design engineering, living documentation, selective helper-file loading, and continuous knowledge evolution.

---

## 🚀 Capabilities Overview

The skillset spans **31 specialized modules** supported by **33 granular, single-concern technical reference & framework checklists** organized into four pillars:

### 1. Orchestration & Lifecycle Workflows
- **`init-project`**: Master entry point to scaffold a project from scratch with phased discovery, stack selection, web essentials audit, and archivist setup.
- **`continue-project`**: Master resume point to audit repo state, inspect backlog items, diagnose blockers, and propose prioritized next tasks.
- **`project-lifecycle`**: Strict 8-phase sequential progression engine (Discovery ➔ Architecture ➔ Governance ➔ Docs ➔ Penpot Design ➔ Scaffolding ➔ Design System ➔ Features).
- **`project-agent-scaffolder`**: Scaffolder for directory-scoped agent instructions (`AGENTS.md`, `GEMINI.md`, and `.agents/rules/`) with 3-tier skill resolution.

### 2. Principal / Staff Specialized Roles & Selective Helper Files
Each role defines core architectural principles in `SKILL.md` and delegates deep framework/domain checklists to granular files in `frameworks/` and `references/` so agents load **only** what is needed for the active project:
- **Strategy & Governance**:
  - `role-product-owner` (INVEST stories, Gherkin BDD, RICE/WSJF prioritization, MVP scope defense)
  - `role-business-analyst` (Ubiquitous Language, EventStorming, state machines, NFR quantification, 8-dimension edge-case matrices)
  - `role-project-manager` (WBS decomposition, critical path DAGs, risk registers, sprint throughput)
  - `role-github-admin` + `references/github-connection-guide.md` (GitHub Projects v2 GraphQL automation, branch rulesets, squash-merge workflows)
- **Architecture & System Design**:
  - `role-software-architect` + `references/architecture-patterns-checklist.md` (Modular Monolith vs Microservices, Hexagonal/Clean boundaries, DDD, Saga/Outbox, OpenTelemetry SLIs/SLOs)
  - `role-tech-researcher` (Time-boxed PoC spikes, weighted trade-off matrices, TCO & lock-in analysis)
  - `role-archivist` (Framework-agnostic `/docs/` taxonomy, MADR 4.0 ADRs, data dictionaries, bidirectional YAML frontmatter cross-linking)
- **Experience & Design Engineering**:
  - `role-ui-ux-designer` & `design-engineering` + `references/ui-craft-and-ergonomics-checklist.md` (Anti-AI-slop rules, fluid typography, OKLCH dark mode, 48x48px touch ergonomics, compositor-only motion, 8 mandatory UI states, Penpot handoffs)
  - `role-copywriter` (Action-oriented microcopy, 3-part error anatomy, empty states, inclusive language, i18n text-expansion budgets)
- **Implementation & Platforms**:
  - `role-frontend-developer` + **7 modular framework & reference guides**:
    - `frameworks/angular.md` (Angular 19/18 Signals, Zoneless, `OnPush`, `@for`/`@defer`, Incremental Hydration, NgRx SignalStore)
    - `frameworks/react.md` (React 19, Next.js 15 App Router, RSC serialization, Server Actions, TanStack Query v5, Zustand v5)
    - `frameworks/vue.md` (Vue 3.5+, Nuxt 3.14+, Composition API, `defineModel`, `shallowRef`, Pinia setup stores, SSR safety)
    - `frameworks/svelte.md` (Svelte 5 Runes `$state`/`$derived`, Snippets, SvelteKit 2 streaming & progressive enhancement)
    - `references/html-and-accessibility-checklist.md` (Semantic landmarks, `<dialog>` & Popover API, mobile form attributes, WCAG 2.2 AA, roving `tabindex`, `<inert>`)
    - `references/css-scss-tailwind-checklist.md` (`@layer`, `@container`, `:has()`, Subgrid, `@starting-style`, logical properties, Dart Sass 1.80+ `@use`/`@forward`, Tailwind v4 `@theme`, Shadow DOM)
    - `references/typescript-and-i18n-checklist.md` (Strict TypeScript 5.7+, `<const T>`, `NoInfer<T>`, discriminated unions, branded types, ICU MessageFormat, `Intl`, RTL, IANA timezone rules)
  - `role-backend-developer` + **5 modular reference & framework guides**:
    - `references/api-design-and-contracts-checklist.md` (OpenAPI 3.1, `Idempotency-Key` locking, RFC 9457 errors, keyset pagination, rate limiting, webhooks, `ETag` conditional caching)
    - `references/database-schema-migrations-and-rls.md` (3NF vs immutable snapshots, UUIDv7, Expand-and-Contract zero-downtime DDL migrations, OCC & `SELECT FOR UPDATE SKIP LOCKED`, partial unique indexes, PostgreSQL RLS)
    - `references/auth-authorization-and-resilience.md` (`__Host-` sessions vs short-lived JWT + rotating refresh tokens, Passkeys & Argon2id, service-layer RBAC/ABAC/ReBAC, Outbox + Idempotent Inbox + DLQ, `SIGTERM` graceful shutdown)
    - `frameworks/node-typescript-backend.md` (Node.js 22 LTS, `AsyncLocalStorage` + Pino trace logging with redaction, `z.strictObject` pipelines, stream backpressure)
    - `frameworks/payload-supabase-pocketbase.md` (Payload CMS 3.x `overrideAccess: false` & transactional `req` hooks, Supabase RLS `(SELECT auth.uid())` optimization, PocketBase WAL transactions)
  - `role-mobile-developer` + **2 modular reference guides**:
    - `references/pwa-service-worker-and-offline-sync.md` (`manifest.webmanifest` split `any`/`maskable` icons, Workbox 7 SW lifecycle, `ChunkLoadError` recovery, IndexedDB/OPFS + SQLite WASM, Outbox conflict resolution LWW/3-Way-Merge/CRDTs)
    - `references/capacitor-native-and-store-compliance.md` (Capacitor 6/7 bridge security, Keychain/Keystore biometrics & iOS uninstall persistence wipe, Universal/App Links, Android 15 Edge-to-Edge safe areas, back-button stack, App Store/Play Store compliance)
- **Verification & Review**:
  - `role-qa-engineer` + `references/testing-pyramid-playwright-checklist.md` (Playwright 1.49+ semantic locators, zero-flakiness rules, `@axe-core/playwright`, visual regression, chaos/boundary matrices)
  - `role-senior-code-reviewer` + `references/pr-code-review-checklist.md` (10-dimension Staff PR audit: architecture, types, concurrency, memory leaks, security, performance, a11y, tests)
- **Security & Regulatory**:
  - `role-security-consultant` + **2 modular security checklists**:
    - `references/owasp-web-api-and-access-checklist.md` (BOLA/IDOR, BFLA, Mass Assignment, SQLi/NoSQLi/XSS/Prototype Pollution/SSTI, SSRF `169.254.169.254`, `__Host-` sessions, PKCE, nonce CSP, file uploads, WebSocket CSWSH security)
    - `references/crypto-supply-chain-and-ai-security.md` (Argon2id, AES-256-GCM envelope encryption & key rotation, CSPRNG, TLS 1.3, lockfile/SBOM supply chain, OWASP Top 10 for LLM 2025)
  - `role-legal-counsel` + `references/gdpr-privacy-compliance-checklist.md` (GDPR Art. 6/9/10, DSAR/Right to Erasure engineering, PII log scrubbing, ePrivacy/cookies, defamation shielding, SPDX license audit)
- **Operations & Discovery**:
  - `role-devops-engineer` + `references/cicd-containers-infra-checklist.md` (SHA-pinned GitHub Actions, OIDC, multi-stage rootless Dockerfiles, Blue-Green/Canary rollouts, RPO/RTO backup drills)
  - `role-performance-consultant` + **10 modular reference & framework guides** (120+ techniques & pitfalls):
    - `references/frontend-html-and-assets-checklist.md` (DOM budgets, resource hints, `fetchpriority`, `<picture>` AVIF/WebP, SVG sprites, video/iframe facades, Speculation Rules API, SSR early flush)
    - `references/frontend-css-scss-less-checklist.md` (Critical CSS, `content-visibility`, CSS containment, compositor motion, `@starting-style`, View Transitions, SCSS/LESS `@mixin` & nesting bloat, zero-CLS font metric overrides)
    - `references/frontend-js-ts-runtime-checklist.md` (Long Tasks `scheduler.yield()` / `postTask()`, layout thrashing, Web Workers & Transferables, 5 memory leak vectors, V8 monomorphism, passive observers, cross-tab `BroadcastChannel`, virtualization, Shadow DOM `adoptedStyleSheets`, cached `Intl` formatters, `crypto.subtle`)
    - `references/frontend-build-bundling-and-delivery.md` (Tree-shaking, barrel file elimination, ES2022 target bundling, CSS code-splitting, JS obfuscation JIT penalty warnings, Brotli q11 / Zstd pre-compression, WASM streaming)
    - `frameworks/angular-performance.md` (Angular 19/18 `OnPush`, Zoneless, Signals, `resource`/`rxResource`, `@for`/`@defer` Incremental Hydration, `NgOptimizedImage`, `runOutsideAngular`)
    - `frameworks/react-nextjs-performance.md` (React 19 Compiler, Context splitting, RSC payload budgets, `React.cache()`, `<Suspense>` streaming, `useTransition`/`useDeferredValue`/`useOptimistic`)
    - `frameworks/vue-nuxt-performance.md` (Vue 3.5+ `shallowRef`, `v-memo`, computed stability, `<KeepAlive :max>`, lazy hydration, Nuxt Islands & `useFetch` payload pruning)
    - `frameworks/svelte-performance.md` (Svelte 5 `$state.raw` vs `$state`, `$derived`, `untrack()`, keyed `{#each}`, SvelteKit streaming & link preloading)
    - `references/backend-api-and-database-performance.md` (N+1/DataLoader, PgBouncer, streaming NDJSON/SSE, event loop health, PostgreSQL 17 indexing, `EXPLAIN (ANALYZE, BUFFERS)`, keyset pagination, HOT updates `fillfactor`)
    - `references/caching-network-and-ci-performance.md` (4-tier caching, stampede defense Singleflight/SWR/Jitter, CDN Cache-Key normalization, Request Hedging, HTTP/3 + `103 Early Hints`, distroless containers, CI performance gates)
  - `role-seo-geo-specialist` + `references/technical-seo-geo-schema-checklist.md` (Crawl budget, partitioned XML sitemaps, canonicals, `hreflang`, `@graph` JSON-LD, Map-to-List HTML parity, Generative Engine Optimization)

### 3. Standards & Protocols
- **`standards-code-style`** + **4 modular tool-agnostic style & lint references**:
  - `references/formatting-naming-and-structure.md` (Whitespace, indentation, quotes, semicolons, braces, cross-language naming matrix, unit suffixes, 5-tier import ordering, cyclomatic/cognitive complexity budgets)
  - `references/javascript-typescript-lint-rules.md` (42 strict static analysis rules across Type Safety, Async/Promises, Immutability, and Operators/Equality)
  - `references/html-css-scss-style-rules.md` (Semantic markup rules, BEM/`kebab-case` selectors, 6-group outside-in CSS property ordering, `rem`/logical units, SCSS `@use`, Tailwind static class maps)
  - `references/sql-backend-and-test-style-rules.md` (PostgreSQL `lower_snake_case` & `pk_`/`fk_`/`uq_`/`chk_`/`idx_` naming, CTE formatting, AAA test structure, factory builders)
- **`standards-omni-role-review`**: Mandatory 360-degree review protocol evaluating PRs and specs across all 14 perspectives.
- **`standards-skills-governance`**: Core governance laws (**Rules A–G**: Project-Agnostic, Tech Helper Files, Extension Protocol, Docs Cross-Linking, English-Only, Zero AI Co-Authorship, and Version Recording & Continuous Knowledge Evolution).
- **`standards-web-essentials`**: Pre-backlog audit checklist covering Sitemaps, Robots, Open Graph, Error Pages, Legal policies, and Security.txt.
- **`standards-git-and-releases`**: Conventional Commits, SemVer, branch protections, and PR release flow.
- **`standards-issue-and-project-management`**: Issue templates, label taxonomies, milestones, and GitHub Projects v2 automation.

### 4. Tooling Philosophy
- **`tooling-free-and-open-source`**: Enforces Free & Open-Source First policy, pairing proprietary standards with free/open-source alternatives (Penpot, PocketBase/Supabase, Playwright, Cloudflare Pages/Workers, Docker).

---

## 🏗️ Recommended Project Structure

To maximize effectiveness when consuming Project Lifecycle Skills, configure your application repository with the following layout:

```text
my-awesome-project/
├── .agents/
│   ├── extensions/                    # 💡 PROJECT EXTENSIONS (Rule C)
│   │   ├── role-frontend-developer.md # Project-specific framework rules & styling
│   │   ├── role-backend-developer.md  # Custom DB schemas, ORM rules, auth logic
│   │   ├── role-legal-counsel.md      # Jurisdiction details, legal disclaimers
│   │   ├── role-seo-geo-specialist.md # Sitemap partitions, local GEO hierarchy
│   │   ├── standards-code-style.md    # Project-specific linter/formatter overrides
│   │   └── standards-git-and-releases.md # Project commit scopes
│   └── rules/                         # Scoped rules referenced by agents
│       ├── git-conventions.md
│       └── mobile-standards.md
│
├── .github/
│   ├── ISSUE_TEMPLATE/                # Managed via standards-issue-and-project-management
│   └── workflows/                     # CI/CD pipelines managed via role-devops-engineer
│
├── docs/                              # 📚 LIVING DOCUMENTATION (Hard Drive Persistence)
│   ├── adr/                           # Architecture Decision Records (role-software-architect)
│   ├── architecture/                  # System blueprints, data models, contracts
│   ├── audits/                        # Web essentials, security, legal, SEO audits
│   ├── research/                      # Spikes, competitor benchmarks, tech trade-offs
│   └── design/                        # Penpot token exports, component hierarchies
│
├── src/                               # Application source code
│   ├── app/ (frontend)
│   │   └── AGENTS.md                  # Frontend-scoped agent directives
│   ├── server/ (backend)
│   │   └── AGENTS.md                  # Backend-scoped agent directives
│   └── shared/
│       └── AGENTS.md                  # Shared types / contracts directives
│
├── tests/
│   └── AGENTS.md                      # QA & E2E testing directives
│
├── AGENTS.md                          # Root agent instructions linking to skills
└── README.md
```

---

## 🛠️ How to Use & Install

### Option A: Global Configuration (User-wide for Antigravity / Agentic IDEs)
Copy the skills into your global agent configuration directory:
```powershell
# Windows
Copy-Item -Path ".\*" -Destination "$HOME\.gemini\config\skills\" -Recurse -Force

# Linux / macOS
cp -r ./* ~/.gemini/config/skills/
```

### Option B: Project-Level Usage
Embed or submodule the skills within your project under `.agents/skills/` or `.github/skills/`, and configure your AI assistant's root instructions (`AGENTS.md` or `.cursorrules`) to reference them.

---

## 🧩 Extending Skills (Without Modifying Base Skills)

Base skills are strictly **project-agnostic** and must never be edited for individual project needs.

To extend or customize rules for your project:
1. Create a file in `<project-root>/.agents/extensions/<skill-name>.md`.
2. Follow the extension template:
   ```markdown
   # role-frontend-developer - Project Extension

   ## Project Context
   Building a mobile-first real-time dashboard using Angular Signals and Tailwind CSS.

   ## Overrides
   - Use Tailwind v4 `@theme` tokens instead of CSS modules.
   - Route slugs use localized paths for public marketing pages.

   ## Additions
   - Charting library: Apache ECharts.
   - State management: NgRx ComponentStore for complex tabular states.
   ```
3. Whenever an agent activates `role-frontend-developer`, it automatically merges your project extension over the base skill instructions.

---

## 📖 Documentation Cross-Linking Protocol

All research, architecture, and design documents in `/docs/` should declare which skills they pertain to using YAML frontmatter:

```markdown
---
skills: [role-archivist, role-backend-developer, role-legal-counsel]
---

# User Data Deletion & Privacy Policy Compliance
...
```

When an agent assumes a role, it queries `/docs/` for relevant documents containing its skill name in the frontmatter, ensuring context is instantly retrieved.

---

## 🤖 Directives for AI Agents

All AI coding assistants (Antigravity, Cursor, Claude Code, GitHub Copilot) operating within this repository or consuming these skills **must follow [AGENTS.md](AGENTS.md)**:
- **Zero AI Co-Authorship (Rule F)**: Never add `Co-authored-by:` or AI signatures to commits unless explicitly requested.
- **Project-Agnosticism (Rule A)**: Never hardcode single-project business logic or URLs into base skills; always provide balanced multi-domain examples.
- **Selective Helper Loading (Rule B)**: Load only the specific `frameworks/` or `references/` checklist needed for the active task to preserve context window efficiency.
- **English-Only (Rule E)**: All skill content must be in English.
- **Hard Drive Persistence (Rule D)**: Persist research, plans, and ADRs directly to `<workspace>/docs/` with `skills: [...]` YAML frontmatter.
- **Version Recording & Continuous Evolution (Rule G)**: Record baseline tool/framework versions in all helper checklists, treat checklists as a quality floor (never a ceiling), and continuously capture project-specific rules in `.agents/extensions/` and universal patterns back into `project-lifecycle-skills`.

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
