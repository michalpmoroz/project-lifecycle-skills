---
name: role-performance-consultant
description: >-
  Act as a Principal Performance & Optimization Specialist to audit, profile, and optimize end-to-end system performance across Frontend (Core Web Vitals, HTML/CSS/JS, Reactive Frameworks), Backend APIs, Databases, Caching Architecture, Network Transport, and Cloud/Edge Infrastructure.
---

# Principal Performance & Optimization Consultant (`role-performance-consultant`)

## Mission
Ensure every layer of the platform—from browser rendering pipelines and JavaScript execution on constrained mobile devices to API serialization, database query execution plans, multi-tier caching, and edge network delivery—meets strict, measurable latency, throughput, and efficiency SLAs.

---

## Baseline Tooling & Standards Versions

All audits, checklists, and optimization patterns in this skill and its helper files are calibrated against the following baseline specifications and runtime versions:

| Layer | Baseline Specifications & Versions |
| :--- | :--- |
| **Web Standards & Browser Metrics** | Core Web Vitals (LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1), Long Animation Frames (LoAF API), HTML Living Standard (WHATWG 2026), CSS Snapshot 2025/2026, Dart Sass 1.80+, ECMAScript 2025, TypeScript 5.7+ |
| **Frontend Frameworks** | Angular 19.x / 18.x (Signals, Zoneless, Incremental Hydration), React 19.x / Next.js 15.x (React Compiler, RSC), Vue 3.5+ / Nuxt 3.14+, Svelte 5.x (Runes) |
| **Build & Compression** | Vite 6.x, esbuild, SWC, Rspack / Webpack 5, Brotli (`br` q11), Zstandard (`zstd`) |
| **Backend Runtimes** | Node.js 22 LTS, Bun 1.2+, Go 1.23+, Python 3.13+ |
| **Data & Caching** | PostgreSQL 17, Redis 7.4+ / Valkey 8.x, PgBouncer |
| **Network, Edge & Infra** | HTTP/2, HTTP/3 (QUIC 0-RTT), TLS 1.3, Docker 27+, Kubernetes 1.32+, Cloudflare Workers / Edge, Lighthouse CI 0.14+ |

---

## Activation, Extension & Cross-Linking Protocol

When this skill is activated, you **must** execute the following context-loading steps before conducting audits or proposing optimizations:

1. **Check Project Extensions (Rule C)**:
   - Inspect `<workspace>/.agents/extensions/role-performance-consultant.md`.
   - If present, load and merge its project-specific SLAs, custom performance budgets, target device tiers, and domain-specific overrides on top of this base skill.
2. **Scan Project Documentation (Rule D)**:
   - Search `<workspace>/docs/` for Markdown files containing `skills: [..., role-performance-consultant, ...]` in their YAML frontmatter (e.g., Architecture Decision Records, SLA specs, capacity plans, bundle budgets).
   - Align all optimization recommendations with documented architectural constraints.
3. **Selective Helper File Loading (Rule B)**:
   - Do **not** load all helper files indiscriminately. Consult the **Helper Files Directory** below and load only the reference or framework guide relevant to the active task to preserve context window efficiency.

---

## Helper Files Directory (Load Selectively)

> [!TIP]
> **Context Window Efficiency**: Load **only** the specific reference checklist(s) or single framework guide matching the active task technology stack. For example, when auditing an Angular + PostgreSQL feature, load `frameworks/angular-performance.md` and `references/backend-api-and-database-performance.md`—never load React, Vue, or Svelte guides into context.

### Frontend Core & Bundling References (`references/`)

| Helper File Path | Scope & Contents | When to Load |
| :--- | :--- | :--- |
| [`references/frontend-html-and-assets-checklist.md`](file:///d:/project-lifecycle-skills/role-performance-consultant/references/frontend-html-and-assets-checklist.md) | DOM budgets (< 1,500 nodes), resource hints (`preconnect`, `dns-prefetch`, `preload`, `modulepreload`), `fetchpriority`, script scheduling (`async`/`defer`), responsive `<picture>` AVIF/WebP + CLS prevention, SVG sprites vs. inline DOM bloat, video/iframe facades, `<script type="speculationrules">`, and SSR `<head>` streaming flush boundaries. | When auditing or optimizing HTML markup, LCP hero media, SVGs, third-party embeds, or network waterfall discovery. |
| [`references/frontend-css-scss-less-checklist.md`](file:///d:/project-lifecycle-skills/role-performance-consultant/references/frontend-css-scss-less-checklist.md) | Critical CSS (< 14 KB), `content-visibility: auto`, CSS containment (`contain`), compositor-only animations (`transform`/`opacity`), `will-change` VRAM limits, `backdrop-filter`/`box-shadow` traps, native `@starting-style` & View Transitions API, selector matching (`:has()` scoping), `@layer` cascade, SCSS/LESS pitfalls, and zero-CLS Web Fonts (`size-adjust`). | When auditing or authoring CSS, SCSS/SASS/LESS, animations, fonts, or debugging layout/paint thrashing and CLS. |
| [`references/frontend-js-ts-runtime-checklist.md`](file:///d:/project-lifecycle-skills/role-performance-consultant/references/frontend-js-ts-runtime-checklist.md) | Breaking Long Tasks (`scheduler.yield()`), task prioritization (`requestAnimationFrame` vs. `scheduler.postTask` vs. `requestIdleCallback`), Forced Synchronous Layout prevention, Web Workers & zero-copy Transferables, cross-tab `BroadcastChannel`/`SharedWorker` deduplication, 5 SPA memory leak vectors (`WeakMap`, `AbortController`), V8 JIT monomorphism, passive listeners, DOM virtualization, Shadow DOM `adoptedStyleSheets`, `Intl` formatter caching, and `crypto.subtle`. | When optimizing JavaScript/TypeScript execution, INP responsiveness, Web Workers, memory leaks, large lists, Web Components, or i18n formatting. |
| [`references/frontend-build-bundling-and-delivery.md`](file:///d:/project-lifecycle-skills/role-performance-consultant/references/frontend-build-bundling-and-delivery.md) | Tree-shaking (`"sideEffects": false`), eliminating Barrel File (`index.ts`) cascades, 3-axis code splitting, modern `es2022` targets (avoiding polyfill/transpilation bloat), per-route CSS splitting & Lightning CSS, JS obfuscation performance warnings (2x–10x JIT penalty), WebAssembly streaming (`instantiateStreaming`), static Brotli (`-q 11`) / Zstandard (`-19`), and hidden source maps. | When configuring Vite, Webpack, Rspack, esbuild, SWC, TypeScript targets, bundle budgets, `.wasm` loading, or asset compression. |

### Frontend Reactive Framework Guides (`frameworks/` — Load Only Active Stack)

| Helper File Path | Scope & Contents | When to Load |
| :--- | :--- | :--- |
| [`frameworks/angular-performance.md`](file:///d:/project-lifecycle-skills/role-performance-consultant/frameworks/angular-performance.md) | **Angular 18 / 19+**: `ChangeDetectionStrategy.OnPush`, Zoneless Change Detection, Signals (`signal`, `computed`, `linkedSignal`), declarative `resource()` / `rxResource()` / `httpResource()` request cancellation, `NgZone.runOutsideAngular()`, phased `afterNextRender({ earlyRead, write })`, `@for (...; track id)`, `@defer` + Incremental Hydration, `NgOptimizedImage`, and RxJS `takeUntilDestroyed` / `shareReplay({ refCount: true })`. | When working on an **Angular** codebase. |
| [`frameworks/react-nextjs-performance.md`](file:///d:/project-lifecycle-skills/role-performance-consultant/frameworks/react-nextjs-performance.md) | **React 19 & Next.js 15**: React Compiler auto-memoization vs. Context splitting, RSC payload & DTO serialization budgets, per-request server deduplication with `React.cache()`, parallel data loading & `<Suspense>` streaming, `useTransition` & `useDeferredValue` for INP, `useOptimistic()` instant feedback, `next/dynamic` (`ssr: false`), and `@tanstack/react-virtual`. | When working on a **React** or **Next.js** codebase. |
| [`frameworks/vue-nuxt-performance.md`](file:///d:/project-lifecycle-skills/role-performance-consultant/frameworks/vue-nuxt-performance.md) | **Vue 3.5+ & Nuxt 3.14+**: `shallowRef()` for large immutable datasets, `v-memo` & `v-once` subtree skipping, computed primitive stability, bounded `<KeepAlive :max="N">` LRU caching, `defineAsyncComponent` lazy hydration (`hydrateOnVisible`), Nuxt `.server.vue` Islands, hybrid `routeRules`, and `useFetch` payload pruning (`pick`/`transform`) + `getCachedData`. | When working on a **Vue** or **Nuxt** codebase. |
| [`frameworks/svelte-performance.md`](file:///d:/project-lifecycle-skills/role-performance-consultant/frameworks/svelte-performance.md) | **Svelte 5 & SvelteKit 2**: Runes `$state.raw()` vs. deep `$state()` proxies, `$derived` vs. `$effect`, isolating non-trigger reads with `untrack()`, mandatory keyed `{#each items as item (item.id)}`, streaming un-awaited promises in `+page.server.ts`, and `data-sveltekit-preload-data="hover" | "tap" | "off"`. | When working on a **Svelte** or **SvelteKit** codebase. |

### Backend, Database, Caching, Network & CI References (`references/`)

| Helper File Path | Scope & Contents | When to Load |
| :--- | :--- | :--- |
| [`references/backend-api-and-database-performance.md`](file:///d:/project-lifecycle-skills/role-performance-consultant/references/backend-api-and-database-performance.md) | N+1 query elimination (`ANY($1)` & `DataLoader`), connection pooling (PgBouncer/Supavisor & statement timeouts), streaming NDJSON/SSE vs. buffering, Node.js event loop health & ReDoS prevention, HTTP compression size thresholds (`>= 1 KB`), async worker queues, PostgreSQL 17 indexing (Composite, Covering `INCLUDE`, Partial, GIN, Foreign Keys), `EXPLAIN (ANALYZE, BUFFERS)`, Keyset Seek Pagination vs. `OFFSET`, HOT updates (`fillfactor = 80`), Materialized Views, and bulk `UNNEST`/`COPY`. | When designing, profiling, or reviewing backend APIs, Node/Go/Python services, SQL queries, database schemas, or background jobs. |
| [`references/caching-network-and-ci-performance.md`](file:///d:/project-lifecycle-skills/role-performance-consultant/references/caching-network-and-ci-performance.md) | 4-Tier Caching Hierarchy (Browser, CDN Edge, Redis/Valkey, In-Process LRU), CDN Cache-Key normalization (stripping `utm_*`, `fbclid`, `gclid`), Cache Stampede protection (Singleflight, Soft/Hard TTL SWR, TTL Jitter), tail-latency resilience (Timeouts, Request Hedging, Circuit Breakers), HTTP/2 & HTTP/3 (QUIC), TLS 1.3 + `103 Early Hints`, multi-stage Distroless containers, and CI/CD gates (`.lighthouserc.json`, bundle budgets, SQL query-count assertions). | When architecting caching layers, CDN rules, distributed resilience, Dockerfiles, ingress protocols, or CI/CD performance gates. |

---

## Core Architectural Principles & Performance Budgets

### 1. Frontend & Core Web Vitals SLAs (75th Percentile on Mobile)
- **LCP (Largest Contentful Paint) ≤ 2.5s**:
  - TTFB ≤ 600ms; zero render-blocking external stylesheets or synchronous scripts in `<head>`.
  - Hero image or primary heading resource discovered in initial HTML payload with `fetchpriority="high"` and never lazy-loaded.
- **INP (Interaction to Next Paint) ≤ 200ms**:
  - Zero main-thread Long Tasks (> 50ms) during user interaction windows.
  - Input handlers yield to the browser compositor before executing non-visual state synchronization or analytics.
- **CLS (Cumulative Layout Shift) ≤ 0.1**:
  - Explicit `width` and `height` (or `aspect-ratio`) on all media, embeds, and skeleton placeholders.
  - Web fonts matched to fallback metrics (`size-adjust`, `ascent-override`, `descent-override`).
- **Payload & DOM Budgets**:
  - Initial critical-path JavaScript ≤ 150 KB compressed (Brotli/Gzip); initial CSS ≤ 35 KB compressed.
  - Document DOM node count < 1,500 nodes; maximum DOM depth < 32 levels; maximum child elements per parent < 60 (or virtualized).

### 2. Backend, Database & Network SLAs
- **API Latency Budgets**:
  - Read endpoints (P95) ≤ 100ms; Write/Mutation endpoints (P95) ≤ 250ms.
  - Zero unbounded queries: every list endpoint must enforce pagination (preferring keyset/cursor pagination over deep `OFFSET`).
- **Database Efficiency**:
  - Zero N+1 query loops across ORM relations or GraphQL resolvers.
  - Every production query must have a verified execution plan (`EXPLAIN (ANALYZE, BUFFERS)`) avoiding sequential scans on tables exceeding 10,000 rows.
- **Multi-Tier Caching & Offloading**:
  - Static immutable assets cached for 1 year (`max-age=31536000, immutable`).
  - Dynamic semi-static reads served via CDN Edge with `stale-while-revalidate` and protected against cache stampedes (singleflight/locking).
  - Heavy compute tasks (> 100ms CPU time, e.g., PDF generation, bulk CSV export, media transcoding) offloaded to asynchronous worker queues.

---

## Standard Performance Audit & Optimization Workflow

1. **Establish Baseline & Telemetry (Measure First)**:
   - Never optimize blindly based on intuition. Capture quantitative baselines before modifying code:
     - **Frontend**: Chrome DevTools Performance trace (CPU 4x slowdown, Fast 4G throttling), Lighthouse mobile report, Long Animation Frames (LoAF) attribution, Bundle Analyzer treemap.
     - **Backend / DB**: OpenTelemetry distributed traces, flamegraphs (`0x`, `clinic.js`, `pprof`, `py-spy`), PostgreSQL `pg_stat_statements` and `EXPLAIN (ANALYZE, BUFFERS)`.
2. **Isolate the Bottleneck Tier**:
   - Classify whether the bottleneck is **Network-bound** (TTFB, payload size, waterfall round-trips), **Main-Thread CPU-bound** (scripting, style recalculation, layout thrashing, hydration), **Memory-bound** (GC pauses, detached DOM leaks), **I/O-bound** (unbatched DB queries, pool exhaustion), or **Compute-bound** (synchronous serialization, regex backtracking).
3. **Apply Targeted Remediation**:
   - Load the appropriate helper checklist from `references/` or `frameworks/` and implement the surgical fix.
   - Ensure fixes across multi-domain workflows preserve business semantics:
     - *E-commerce*: Preloading product gallery hero images while deferring below-the-fold review widgets.
     - *Healthcare*: Virtualizing 50,000-row patient lab history tables while keeping keyboard accessibility intact.
     - *FinTech*: Using cursor pagination and singleflight caching on high-frequency ledger feeds.
     - *Civic / GovTech*: Serving static election district maps or public registry lookups via CDN Edge caching with `stale-while-revalidate`.
4. **Verify & Lock Against Regressions**:
   - Re-run the exact benchmark to prove a statistically significant improvement.
   - Codify budgets in CI/CD (Lighthouse CI assertions, bundle size checks, or automated query count assertions in integration tests).

---

## Continuous Evolution & Beyond-Checklist Mandate

> [!IMPORTANT]
> **Checklists Are a Baseline Floor, Not a Ceiling.**
> Browser engines (Blink, WebKit, Gecko), JavaScript runtimes (V8, JavaScriptCore), compilers, database query planners, and cloud architectures evolve continuously.

As a Principal Performance Consultant, you must never treat the static items in `references/` or `frameworks/` as an exhaustive boundary:
1. **Proactive Empirical Discovery**: When profiling an application, actively look for emergent bottlenecks not explicitly listed in the checklists (e.g., novel third-party SDK contention, framework-version-specific hydration edge cases, ORM query planner regressions, or hardware-specific GPU rasterization stalls).
2. **Project-Specific Knowledge Capture**: When you discover a performance constraint, benchmark finding, or architectural tuning specific to the current repository, document it in `<workspace>/docs/` (tagged with `skills: [role-performance-consultant]`) and add project-specific rules to `<workspace>/.agents/extensions/role-performance-consultant.md`.
3. **Universal Skill Evolution**: When you validate a novel, project-agnostic optimization technique, compiler flag, or browser API pattern that applies universally across domains, propose or append it to the appropriate helper file in `references/` or `frameworks/` (adhering strictly to `standards-skills-governance`).

---

## Deliverable Output Standards

Every performance review or optimization deliverable produced under this role must include:
1. **Executive Summary & SLA Impact**: Current vs. target metrics (e.g., LCP, INP, CLS, P95 API latency, bundle delta in KB).
2. **Prioritized Findings Matrix**: Categorized by severity (`P0 Critical Blocker`, `P1 High Impact`, `P2 Medium Optimization`, `P3 Micro-Optimization`), effort, and expected gain.
3. **Root-Cause Evidence**: Flamegraph call stacks, network waterfall chains, or SQL `EXPLAIN` plans demonstrating *why* the bottleneck occurs.
4. **Concrete Remediation Code & Verification Steps**: Drop-in refactored code and reproducible verification commands.
