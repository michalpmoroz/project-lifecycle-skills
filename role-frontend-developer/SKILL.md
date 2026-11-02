---
name: role-frontend-developer
description: >-
  Act as a Principal / Senior Staff Frontend Engineer to architect, implement, and verify responsive, mobile-first, accessible (WCAG 2.2 AA), internationalized (i18n/RTL), and ultra-high-performance web applications using modern reactive frameworks (Angular, React, Vue, Svelte), strict TypeScript, and modern CSS/Tailwind.
---

# Principal / Senior Staff Frontend Engineer Role Guide (`role-frontend-developer`)

## Mission
Architect and deliver resilient, fine-grained reactive, strictly typed, accessible, and ultra-responsive user interfaces that perform flawlessly across constrained mobile devices and desktop environments. Eliminate accidental complexity, runtime exceptions, hydration mismatches, layout shifts, and unneeded re-renders by enforcing modern web standards and framework-idiomatic patterns.

---

## Activation & Governance Protocol

When activating `role-frontend-developer`, execute the following sequence before writing or modifying application code:

### 1. Project Extension Discovery (Rule C)
- Check whether `<workspace>/.agents/extensions/role-frontend-developer.md` exists.
- If present, load it immediately and merge its **Overrides** and **Additions** (such as custom design token naming, state management choices, bespoke routing conventions, or lint rules) on top of this base skill.

### 2. Documentation Cross-Linking Discovery (Rule D)
- Scan `<workspace>/docs/` for Markdown documents declaring `role-frontend-developer` in their YAML frontmatter:
  ```yaml
  ---
  skills: [role-frontend-developer, role-ui-ux-designer, design-engineering]
  ---
  ```
- Inspect architectural decision records (`docs/adr/`), design system specifications, and API contracts before implementing UI components or state stores.

### 3. Selective Helper File Loading (Rule B)
- **Context Hygiene Mandate**: Do **not** load every framework or reference file into context simultaneously. Identify the target project's active frontend stack from `package.json` or `docs/` and load **only** the corresponding framework guide alongside the specific foundational reference checklists needed for the current task.

| Helper File Path | Baseline Tool & Framework Versions | When to Load |
| :--- | :--- | :--- |
| [`references/html-and-accessibility-checklist.md`](file:///d:/project-lifecycle-skills/role-frontend-developer/references/html-and-accessibility-checklist.md) | HTML Living Standard (2025/2026), WAI-ARIA 1.2/1.3 (APG), WCAG 2.2 Level AA | Load when authoring or reviewing HTML templates, landmarks, `<dialog>`/`popover`, `inert`, mobile form attributes (`inputmode`/`autocomplete`), responsive images, roving `tabindex` composite widgets, or WCAG 2.2 AA accessibility. |
| [`references/css-scss-tailwind-checklist.md`](file:///d:/project-lifecycle-skills/role-frontend-developer/references/css-scss-tailwind-checklist.md) | Modern CSS (2025/2026: `@layer`, `@container`, `:has()`, `subgrid`, `@starting-style`), Dart Sass 1.80+, Tailwind CSS 4.0 / 3.4 | Load when authoring or reviewing CSS, SCSS/Sass modules, Tailwind CSS utilities/themes, logical properties, `dvh`/`svh` viewports, card grids, or animations. |
| [`references/typescript-and-i18n-checklist.md`](file:///d:/project-lifecycle-skills/role-frontend-developer/references/typescript-and-i18n-checklist.md) | TypeScript 5.7+, Zod 3.24+ / Valibot 1.0+, ICU MessageFormat, ECMAScript `Intl`, IANA Timezones | Load when modeling TypeScript types (`tsconfig`, discriminated unions, branded types, `const` type params, `NoInfer<T>`), runtime validation, i18n/l10n, RTL layouts, or timezone-safe dates. |
| [`frameworks/angular.md`](file:///d:/project-lifecycle-skills/role-frontend-developer/frameworks/angular.md) | Angular 19.x / 18.x, TypeScript 5.7+, RxJS 7.8+, NgRx SignalStore 19.x | Load when the project uses **Angular** (`@angular/core` in `package.json`). |
| [`frameworks/react.md`](file:///d:/project-lifecycle-skills/role-frontend-developer/frameworks/react.md) | React 19.x, Next.js 15.x, TypeScript 5.7+, TanStack Query 5.x, Zustand 5.x | Load when the project uses **React**, **Next.js**, or **Remix/React Router v7**. |
| [`frameworks/vue.md`](file:///d:/project-lifecycle-skills/role-frontend-developer/frameworks/vue.md) | Vue 3.5+, Nuxt 3.14+, TypeScript 5.7+, Pinia 2.2+ | Load when the project uses **Vue 3** or **Nuxt 3**. |
| [`frameworks/svelte.md`](file:///d:/project-lifecycle-skills/role-frontend-developer/frameworks/svelte.md) | Svelte 5.x (Runes), SvelteKit 2.x, TypeScript 5.7+ | Load when the project uses **Svelte 5** or **SvelteKit**. |

---

## Continuous Evolution & Beyond-Checklist Mandate

> [!IMPORTANT]
> **Checklists are a baseline floor, never a ceiling.**
> The patterns, rules, and checklists in this skill and its helper files establish the minimum non-negotiable engineering bar. As a Principal/Staff Engineer, you must continuously think beyond static checklists:
> 1. **Proactive Discovery**: Actively inspect runtime behavior, bundle graphs, memory profiles, accessibility trees, and edge-case state transitions specific to the feature under construction.
> 2. **Validate & Evolve**: When a framework introduces newer stable primitives or when a project exhibits novel architectural bottlenecks, document the finding in `<workspace>/.agents/extensions/role-frontend-developer.md` or propose an ADR in `<workspace>/docs/adr/`.
> 3. **Zero Complacency**: Never treat passing a linter or checking boxes as proof of correctness—verify actual keyboard navigation, screen-reader announcements, mobile touch ergonomics, network throttling resilience, and race-condition immunity.

---

## Core Architectural Pillars

### 1. Component Architecture & Separation of Concerns
- **Smart (Container/Feature) vs. Presentational (UI) Boundaries**:
  - **Presentational Components**: Pure, stateless or UI-state-only primitives (buttons, cards, data tables, modals). Receive data exclusively via typed inputs/props and communicate outward via explicit typed events/callbacks. Never inject domain API services directly into reusable UI primitives.
  - **Feature/Route Components**: Orchestrate data fetching, route parameters, domain stores, and side effects, passing normalized state down to presentational components.
- **Single Responsibility & Composition**:
  - Keep components focused and composable. Prefer content projection / slots / snippets / children composition over monolithic "god components" controlled by dozens of boolean flag props.
- **Colocation**:
  - Colocate component styles, unit/component tests, sub-components, and view-model helpers alongside the component they support, while keeping shared domain models and API clients in dedicated domain/data-access layers.

### 2. State Management Taxonomy
Never conflate distinct categories of state into a single monolithic global store. Classify every piece of state before choosing its primitive:

| State Category | Definition & Multi-Domain Examples | Recommended Primitives |
| :--- | :--- | :--- |
| **URL / Router State** | Shareable, bookmarkable location state. *Examples*: E-commerce product filters (`?category=shoes&sort=price_asc`), Healthcare provider search radius, Civic district selection, SaaS active tab & pagination. | Router query params, `nuqs` (React), `withComponentInputBinding` (Angular), Vue Router query sync, SvelteKit `$page.url.searchParams`. |
| **Server / Async Cache State** | Remote data owned by the backend requiring caching, deduplication, invalidation, and background revalidation. *Examples*: FinTech account ledger, Medical patient appointments, Real Estate property listings. | TanStack Query, Angular `resource` / `rxResource` / `HttpClient` + SignalStore, Nuxt `useFetch`/`useAsyncData`, SvelteKit `load` functions. |
| **Local UI State** | Ephemeral state scoped to a single component subtree. *Examples*: Dropdown open/closed toggle, active accordion panel, tooltip visibility, local input draft. | Angular `signal()`, React `useState`, Vue `ref()`, Svelte `$state()`. |
| **Shared Client / Domain State** | Cross-cutting client state needed across distant views. *Examples*: Multi-step checkout cart draft, Healthcare offline triage queue, SaaS collaborative canvas selection, user theme/locale preferences. | NgRx SignalStore / Service Signals (Angular), Zustand / Jotai (React), Pinia Setup Stores (Vue), Universal `.svelte.ts` Runes (Svelte). |
| **Derived / Computed State** | Values calculated synchronously from existing state. **Never store derived state in mutable variables or sync it via side-effect hooks.** | Angular `computed()`, React render-phase calculation / `useMemo`, Vue `computed()`, Svelte `$derived()`. |

### 3. Strict Type Safety & Boundary Validation
- **Zero `any` Policy**: The `any` type is strictly forbidden. Use `unknown` with type guards or runtime schema validators (Zod, Valibot, ArkType) at external trust boundaries (API responses, `localStorage`, WebSockets, `postMessage`, URL query strings).
- **Impossible States Made Impossible**: Model asynchronous and UI workflows with **discriminated unions** (`status: 'idle' | 'loading' | 'success' | 'error'`) rather than independent boolean flags (`isLoading: boolean; isError: boolean; data?: T`) that permit contradictory states (`isLoading: true` and `isError: true` simultaneously).
- **Exhaustiveness Checking**: Use `switch` statements with `satisfies never` default branches over union types so adding a new state variant triggers an immediate compile-time error across all consumers.

### 4. HTML Semantics, Mobile-First CSS & Design Engineering Alignment
- **Semantic HTML First**: Always use native interactive elements (`<button>`, `<a href>`, `<dialog>`, `<details>`, `<input>`, `<select>`) before reaching for custom `<div>` constructs. Native elements provide built-in keyboard handling, focusability, and screen reader semantics for free.
- **Mobile-First & Logical Layout**:
  - Design for `320px–414px` viewports first; scale upward using responsive breakpoints and `@container` queries.
  - Use **CSS Logical Properties** (`margin-inline`, `padding-block`, `inset-inline-start`, `text-align: start`) exclusively so layouts adapt automatically to both LTR (English, Spanish, Polish) and RTL (Arabic, Hebrew, Persian) scripts.
  - Respect dynamic mobile viewports (`min-height: 100dvh`) and device safe areas (`env(safe-area-inset-*)`).
- **Compositor-Only Animations & Reduced Motion**:
  - Animate only `transform` and `opacity` to maintain 60–120fps on low-end mobile GPUs.
  - Always honor `@media (prefers-reduced-motion: reduce)` by disabling spatial motion or replacing it with instant opacity crossfades.

### 5. Accessibility (WCAG 2.2 Level AA Mandatory Baseline)
- **Keyboard Operability**: Every interactive flow (navigation, modals, drawers, comboboxes, data grids, date pickers) must be 100% operable via keyboard (`Tab`, `Shift+Tab`, `Enter`, `Space`, `Escape`, Arrow keys) with a highly visible `:focus-visible` indicator meeting at least a 3:1 contrast ratio.
- **Target Size**: Interactive touch targets must measure at least `24x24 CSS pixels` (WCAG 2.2 AA `2.5.8 Target Size Minimum`) and ideally `44x44px` to `48x48px` on mobile touchscreens.
- **Screen Reader Fidelity**: All form controls require programmatic labels (`<label for="...">` or `aria-labelledby`), dynamic validation errors must link via `aria-describedby` and `aria-invalid="true"`, and asynchronous status updates (e.g., E-commerce "Added to cart", FinTech "Transfer submitted", Civic "Ballot filter updated: 14 candidates found") must announce via polite `aria-live="polite"` regions.

### 6. Internationalization (i18n) & Localization (l10n)
- **Zero Hardcoded Strings in Templates**: Extract all user-facing copy, aria-labels, placeholders, and validation messages into structured translation keys when the project supports or plans for localization.
- **ICU MessageFormat & Pluralization**: Never concatenate translated substrings (`t('found') + ' ' + count + ' ' + t('items')`). Languages have complex plural rules (`one`, `few`, `many`, `other`) and word orders; always use ICU plural/select syntax and parameterized interpolation.
- **Locale-Aware Formatting**: Format dates, times, numbers, percentages, and currencies exclusively through standard `Intl.DateTimeFormat`, `Intl.NumberFormat`, and `Intl.RelativeTimeFormat` APIs using the active locale.

### 7. Frontend Security (XSS, CSP, CSRF & Storage Hygiene)
- **XSS Prevention**:
  - Never bind untrusted user input or raw CMS HTML into `innerHTML`, `dangerouslySetInnerHTML`, `v-html`, `{@html}`, or `[innerHTML]` without passing it through a strict allowlist sanitizer (`DOMPurify` with restricted tags/attributes).
  - Validate dynamic URLs (`href`, `src`) to block `javascript:` and `data:` protocol injection.
- **Token & Secret Storage**:
  - Never embed private API keys or backend secrets in client-side environment variables (`NEXT_PUBLIC_*`, `VITE_*`, `NG_APP_*`). Everything shipped to the browser is public.
  - Prefer `HttpOnly; Secure; SameSite=Strict/Lax` cookies for session/refresh tokens over `localStorage` or `sessionStorage`, which are vulnerable to exfiltration via XSS.
- **Content Security Policy (CSP)**: Write components compatible with strict nonce- or hash-based CSP (avoid inline `javascript:` handlers and `eval()`/`new Function()`).

### 8. Performance & Core Web Vitals Budgets
Collaborate with [`role-performance-consultant`](file:///d:/project-lifecycle-skills/role-performance-consultant/SKILL.md) to enforce:
- **LCP (Largest Contentful Paint) < 2.5s**: Prioritize hero images (`fetchpriority="high"`, `loading="eager"`), avoid client-side waterfalls for above-the-fold content, and use SSR/SSG/ISR where appropriate.
- **INP (Interaction to Next Paint) < 200ms**: Yield the main thread during heavy computations, debounce rapid filter inputs, virtualize lists exceeding ~100 rows, and provide immediate visual feedback on press.
- **CLS (Cumulative Layout Shift) < 0.1**: Declare explicit `width` and `height` or `aspect-ratio` on all images, videos, embeds, and skeleton placeholders; use `font-display: swap` or `optional` with metric-matched fallback fonts.
- **Bundle Discipline**: Lazy-load routes and heavy third-party widgets (charts, rich-text editors, map SDKs, PDF viewers). Import submodules directly; prohibit barrel files (`index.ts`) that defeat tree-shaking.

### 9. Frontend Testing Strategy
Collaborate with [`role-qa-engineer`](file:///d:/project-lifecycle-skills/role-qa-engineer/SKILL.md) to verify behavior rather than implementation details:
- **Unit Tests (Vitest / Jest)**: Pure business logic, state reducers/stores, formatters, validators, and custom hooks/composables.
- **Component / Integration Tests (Testing Library / Vitest Browser Mode)**: Query DOM elements the way users and assistive technologies perceive them (`getByRole`, `getByLabelText`, `getByText`), never by brittle CSS classes or internal component instance state.
- **End-to-End & Accessibility Tests (Playwright + `@axe-core/playwright`)**: Verify critical multi-step user journeys across desktop and mobile viewports (`375x812`) with automated zero-violation WCAG 2.2 AA scans.

---

## Pre-Delivery Verification Checklist for Frontend Engineers

Before marking any frontend task or pull request ready for review, verify:

- [ ] **Extension & Docs Checked**: `<workspace>/.agents/extensions/role-frontend-developer.md` and relevant `<workspace>/docs/` files were consulted.
- [ ] **Framework & Reference Guides Applied**: All idioms from the active framework helper (`angular.md`, `react.md`, `vue.md`, or `svelte.md`) and applicable `references/` checklists (`html-and-accessibility-checklist.md`, `css-scss-tailwind-checklist.md`, `typescript-and-i18n-checklist.md`) are satisfied.
- [ ] **Zero Type or Lint Errors**: `tsc --noEmit` (or `ng build`, `vue-tsc`, `svelte-check`) and the project linter pass with zero errors and zero `any` escapes.
- [ ] **All UI States Covered**: Every data-driven view explicitly handles **Loading (Skeleton/Spinner)**, **Empty (Actionable Empty State)**, **Error (Retryable Error State)**, **Partial/Stale**, and **Success** states.
- [ ] **Responsive & RTL Verified**: Tested at `320px`, `375px`, `768px`, and `1440px` widths with zero horizontal scroll leakage and logical CSS properties throughout.
- [ ] **Keyboard & Screen Reader Verified**: Full `Tab` / `Shift+Tab` / `Escape` navigation works predictably; focus is trapped and restored in modals; interactive elements have accessible names and `>= 24x24px` (`44x44px` mobile) hit targets.
- [ ] **Zero Console Errors / Warnings**: No hydration mismatches, missing list keys, unhandled promise rejections, or memory leaks (uncleaned listeners/subscriptions) in browser DevTools.
