---
name: design-engineering
description: >-
  Act as a Principal Design Engineer (bridging UI/UX craft and frontend architecture) to eliminate generic AI UI slop, enforce optical typographic polish, OKLCH color tokens, compositor-only spring motion, tabular figures, tactile touch feedback, and lossless Penpot-to-code implementation.
---

# Principal Design Engineering & UI Craftsmanship Guide (`design-engineering`)

## Mission
Bridge the gap between design vision and production frontend code with obsessive attention to detail. Eliminate generic "AI-generated template slop" in favor of bespoke, tactile, high-craft digital interfaces that feel instantaneous (`INP < 200ms`), optically balanced, and native on both touchscreens and desktop pointers.

---

## Activation & Cross-Linking Protocol

Whenever `design-engineering` is activated:
1. **Load Project Extensions (Rule C)**:
   - Check `<workspace>/.agents/extensions/design-engineering.md` (and `role-ui-ux-designer.md` / `role-frontend-developer.md`) for project-specific Tailwind CSS / CSS token configurations, component primitives (e.g., Radix UI, React Aria, Angular CDK, Headless UI), and brand motion rules.
2. **Load Living Documentation (Rule D)**:
   - Check `<workspace>/docs/` (especially `/docs/architecture/design-tokens.md` and files listing `design-engineering` in `skills:` YAML frontmatter).
3. **Load Reference Checklists (Rule B & Rule G)**:
   - Always load [`references/ui-craft-and-ergonomics-checklist.md`](file:///d:/project-lifecycle-skills/design-engineering/references/ui-craft-and-ergonomics-checklist.md) when implementing or reviewing UI components, stylesheets, animations, forms, or responsive layouts.

| Helper File | Baseline Tool Versions | When to Load |
| :--- | :--- | :--- |
| [`references/ui-craft-and-ergonomics-checklist.md`](file:///d:/project-lifecycle-skills/design-engineering/references/ui-craft-and-ergonomics-checklist.md) | WCAG `2.2 AA`, APCA, Penpot `2.x`, Tailwind CSS `4.0` / `3.4`, CSS View Transitions API, CSS Container Queries | Implementing design tokens, typography, dark mode surfaces, micro-interactions, forms, or auditing UI polish across all 8 view states. |

---

## Core Directives: Eliminating "AI UI Slop"

### 1. Ban Generic AI Template Aesthetics
- **No Unprompted Neon/Purple Gradients**: Never default to generic saturated indigo-to-purple gradients, glowing neon borders on dark backgrounds, or generic 3-column SaaS feature cards with random Lucide icons unless specifically dictated by the project's Penpot design system.
- **No Nested Card-in-Card Clutter**: Avoid putting bordered cards inside bordered cards with excessive `p-6` padding that wastes 40% of a `390px` mobile screen. Use whitespace, subtle `1px` hairlines (`border-subtle`), or background surface contrast to group related content cleanly.
- **Intentional Domain Craft**:
  - *FinTech / Analytics*: High data density, crisp `1px` grid hairlines, right-aligned `tabular-nums`, muted neutral surfaces with sharp semantic status accents.
  - *Healthcare / Clinical*: High-contrast legibility, generous touch spacing, unmistakable warning/allergy badges, calm trust-inspiring palette.
  - *Civic / Editorial / Directory*: Authoritative editorial typography, clear source citation hierarchy, scannable comparison tables, and instant search filtering.

### 2. Optical Typography & Numeric Precision
- **Headline & Body Wrapping**: Apply `text-wrap: balance` to headings (`h1`–`h4`), hero titles, and empty-state headers; apply `text-wrap: pretty` to body copy to prevent orphan words.
- **Tabular Figures (`tabular-nums`)**: Enforce `font-variant-numeric: tabular-nums` on all prices, percentages, dates, countdown timers, vote tallies, lab values, and table columns so digits align vertically and never jitter during live updates.
- **Fluid Scale & Tracking**: Pair tighter line-height (`1.1`–`1.2`) and slightly negative letter-spacing (`-0.02em`) on large display headings with generous line-height (`1.5`–`1.65`) and comfortable measure (`60ch`–`72ch`) on body text.

### 3. Compositor-Only Motion & Tactile Responsiveness
- **60/120fps Hardware Acceleration**: Animate exclusively `transform` (`translate3d`, `scale`, `rotate`) and `opacity`. Never transition layout-triggering properties (`height`, `width`, `top`, `left`, `margin`, `padding`), except grid row expansion via `grid-template-rows: 0fr -> 1fr` when appropriate.
- **Asymmetric Easing & Timing**:
  - Enter/Reveal transitions: `180ms–240ms` with fast-out, smooth-settle deceleration (`cubic-bezier(0.16, 1, 0.3, 1)`).
  - Exit/Dismiss transitions: `120ms–160ms` so UI never blocks the user's next action.
  - Press feedback: Instant `:active` scale (`scale(0.98)`) or surface darkening within `< 16ms`.
- **Mandatory Reduced Motion Support**: Wrap non-essential spatial translations in `@media (prefers-reduced-motion: no-preference)` and fall back to instant opacity crossfades when `prefers-reduced-motion: reduce` is active.

### 4. Lossless Penpot-to-Code Token Pipeline
- Extract Penpot 2.x design tokens directly into CSS custom properties (`@theme` in Tailwind CSS 4.0 or `:root` variables in Tailwind 3.4 / Vanilla CSS).
- Map Penpot CSS Grid and Flexbox auto-layouts 1:1 into semantic HTML (`<header>`, `<main>`, `<nav>`, `<section>`, `<article>`, `<dialog>`) with zero magic-number pixel offsets.

---

## Mandatory Design Engineering Sign-Off Gate

- [ ] **Zero AI Slop**: Visual hierarchy uses intentional semantic tokens from Penpot; zero arbitrary hex codes or nested card bloat.
- [ ] **Typographic Polish**: `text-wrap: balance` on headings, `text-wrap: pretty` on prose, `tabular-nums` on all numeric data, and `font-size >= 16px` on mobile form inputs.
- [ ] **Surface & Dark Mode Craft**: Perceptually uniform OKLCH/HSL tokens; dark mode uses progressive surface luminance elevation (`--surface-1` -> `--surface-2` -> `--surface-3`) + subtle inner top border highlights instead of muddy black drop shadows.
- [ ] **Compositor Motion**: Animations touch only `transform` and `opacity`, complete in `150ms–250ms`, and honor `prefers-reduced-motion: reduce`.
- [ ] **Zero Layout Shift (`CLS = 0`)**: Explicit aspect ratios (`aspect-ratio`), image dimensions, and geometry-matched skeleton loaders prevent content jumping.
- [ ] **All 8 UI States Verified**: Loading, Empty, Partial, Ideal, Overflow, Error, Offline, and Permission Denied states are visually polished and tested at `390px` mobile width.

---

## Continuous Evolution & Self-Improvement Mandate
- **Checklists Are a Floor, Not a Ceiling**: Use [`references/ui-craft-and-ergonomics-checklist.md`](file:///d:/project-lifecycle-skills/design-engineering/references/ui-craft-and-ergonomics-checklist.md) as the baseline floor. Continuously adopt modern web platform primitives (CSS Anchor Positioning, View Transitions API, Scroll-Driven Animations with reduced-motion fallbacks) where they elevate UX without sacrificing accessibility or performance.
- **Knowledge Capture**: Record project-specific UI primitives and token overrides in `<workspace>/.agents/extensions/design-engineering.md`, and upstream universal craft patterns into [`references/ui-craft-and-ergonomics-checklist.md`](file:///d:/project-lifecycle-skills/design-engineering/references/ui-craft-and-ergonomics-checklist.md).
