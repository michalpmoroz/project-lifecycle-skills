# Modern CSS, SCSS/Sass & Tailwind CSS Engineering Checklist (`references/css-scss-tailwind-checklist.md`)

> **Baseline Versions**: Modern CSS (Baseline 2025/2026: `@layer`, `@container`, `:has()`, `subgrid`, `@starting-style`), Dart Sass `1.80+`, Tailwind CSS `4.0` / `3.4`, Web Components Shadow DOM (`::part`, `adoptedStyleSheets`).
> **Mandate**: Load and apply this checklist whenever authoring stylesheets, design tokens, layout grids, responsive components, or animations.

> [!NOTE]
> **Continuous Evolution Mandate**: This checklist is a baseline quality floor, never a ceiling. Proactively leverage native browser layout and animation primitives to eliminate fragile JavaScript measurement loops and bundle bloat.

---

## 1. Modern CSS Baseline Capabilities (2025/2026)

### 1.1 Cascade Layers, Container Queries & Relational Selectors
- **Cascade Layers (`@layer`)**: Structure deterministic specificity without `!important` wars: `@layer reset, tokens, base, components, utilities;`.
- **Container Queries (`@container`)**: Make reusable cards and widgets responsive to their parent slot width rather than global viewport width:
  ```css
  .widget-host {
    container-type: inline-size;
    container-name: card;
  }
  @container card (min-width: 420px) {
    .widget-layout { grid-template-columns: 160px 1fr; }
  }
  ```
- **Relational Selector (`:has()`)**: Style parent containers based on child states without JavaScript:
  ```css
  /* Highlight form group when input inside has validation error */
  .field-group:has(input:user-invalid) {
    border-color: var(--color-danger);
  }
  ```

### 1.2 CSS Subgrid (`grid-template-rows: subgrid`) for Aligned Card Grids
When rendering a multi-column grid of cards with variable-length titles, descriptions, metadata badges, and action footers (e.g., E-commerce product cards, SaaS pricing tiers, Civic candidate profiles, Healthcare clinic cards), use `subgrid` so internal card rows align horizontally across every card in the row:

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(min(100%, 280px), 1fr));
  /* Define 4 explicit tracks per card: Media, Title, Body, Footer */
  grid-auto-rows: auto;
  gap: 1.5rem;
}

.card-item {
  /* Span 4 rows of the parent grid and inherit track sizing via subgrid */
  grid-row: span 4;
  display: grid;
  grid-template-rows: subgrid;
  gap: 0.75rem;
}
```

### 1.3 Native Top-Layer Entry & Exit Animations (`@starting-style` & `allow-discrete`)
Animate native `<dialog>` modals and `[popover]` menus cleanly on both entry and exit using `@starting-style` and `transition-behavior: allow-discrete`—without keeping hidden DOM nodes mounted or adding heavy JS animation libraries:

```css
dialog,
[popover] {
  opacity: 0;
  transform: translateY(8px) scale(0.98);
  transition:
    opacity 180ms cubic-bezier(0.16, 1, 0.3, 1),
    transform 180ms cubic-bezier(0.16, 1, 0.3, 1),
    overlay 180ms allow-discrete,
    display 180ms allow-discrete;
}

/* Open state */
dialog[open],
[popover]:popover-open {
  opacity: 1;
  transform: translateY(0) scale(1);
}

/* Entry keyframe when transitioning from display: none to open */
@starting-style {
  dialog[open],
  [popover]:popover-open {
    opacity: 0;
    transform: translateY(8px) scale(0.98);
  }
}

@media (prefers-reduced-motion: reduce) {
  dialog,
  [popover] {
    transition-duration: 0.01ms;
    transform: none;
  }
}
```

### 1.4 Logical Properties, Dynamic Viewports & Typography Polish
- **CSS Logical Properties (Mandatory for i18n/RTL)**:
  - Replace `margin-left` / `margin-right` with `margin-inline-start` / `margin-inline-end` (or `margin-inline`).
  - Replace `padding-top` / `padding-bottom` with `padding-block-start` / `padding-block-end` (or `padding-block`).
  - Replace `left` / `right` with `inset-inline-start` / `inset-inline-end`.
  - Replace `text-align: left` / `right` with `text-align: start` / `end`.
- **Dynamic Viewport Units**: Use `100dvh` (dynamic viewport height) or `100svh` (small viewport height) instead of `100vh` so mobile browser address bars do not clip sticky footers or modal action buttons.
- **Typography Polish**:
  - `text-wrap: balance` on headings (`h1`–`h4`) to eliminate single-word orphans.
  - `text-wrap: pretty` on body paragraphs.
  - `font-variant-numeric: tabular-nums` on all prices, financial ledgers, countdown timers, election vote tallies, and table columns so digits align vertically without jitter.

---

## 2. Dart Sass / SCSS Discipline (Sass `1.80+`)

When a project uses `.scss`:
- **`@use` and `@forward` Only**: `@import` is officially deprecated in Dart Sass 1.80+ and will be removed. Always use modular `@use 'tokens' as t;` and `@forward`.
- **Use Built-In Modules**: Use `@use 'sass:math';`, `@use 'sass:color';`, and `@use 'sass:map';` (e.g., `color.adjust()` or native CSS `color-mix()` instead of deprecated global `darken()`/`lighten()`).
- **Zero CSS Duplication inside `@use`d Partials**: Shared `.scss` files imported via `@use` across multiple components must contain **only** compile-time mixins, functions, or `%placeholders`—never raw CSS selector rules, which would be duplicated into every importing component's stylesheet bundle!
- **Prefer CSS Custom Properties (`--var`) Over `$scss-vars` for Theme Tokens**: Compile-time `$scss` variables cannot respond to dark mode (`prefers-color-scheme`), high-contrast mode, or tenant runtime branding without duplicating selectors.
- **Max Nesting Depth <= 3**: Never nest selectors deeper than 3 levels; prefer flat BEM or component-scoped classes.

---

## 3. Tailwind CSS (v4.0 & v3.4) Engineering

- **Tailwind CSS v4 CSS-First Configuration**: In Tailwind v4 projects, configure design tokens directly in CSS via `@import "tailwindcss";` and `@theme { --color-brand-primary: oklch(0.55 0.18 250); }`.
- **Deterministic Class Merging (`cn`)**: Always combine `clsx` and `tailwind-merge` when building reusable UI components that accept custom `className` overrides:
  ```ts
  import { clsx, type ClassValue } from 'clsx';
  import { twMerge } from 'tailwind-merge';

  export function cn(...inputs: ClassValue[]): string {
    return twMerge(clsx(inputs));
  }
  ```
- **No Dynamic String Interpolation of Tailwind Classes**:
  - **BROKEN**: ``className={`text-${color}-600 bg-${color}-100`}`` (Tailwind's static scanner cannot detect dynamically concatenated class names and will purge them from production builds).
  - **CORRECT**: Map variants using a static lookup dictionary or `cva` (Class Variance Authority) with full literal class strings.

---

## 4. Shadow DOM & Web Components Encapsulation

- Style the host element via `:host { display: block; }` and `:host([disabled])`.
- Expose intentional styling hooks across Shadow boundaries via CSS Custom Properties (`var(--card-padding, 1rem)`) and `part="button"` (`::part(button)`).
- Share constructable stylesheets across Web Component instances via `shadowRoot.adoptedStyleSheets = [sharedSheet]` so the browser parses the CSS AST once in memory rather than cloning `<style>` tags into every shadow root.
