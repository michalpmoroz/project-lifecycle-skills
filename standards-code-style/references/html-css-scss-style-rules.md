# HTML, CSS, SCSS/LESS & Tailwind CSS Style Rules Catalog

> **Baseline Specification & Tooling Versions (Rule G)**
> - **Web Platform Standards**: WHATWG HTML Living Standard (`2026`), W3C CSS Snapshot `2025/2026`, WCAG `2.2 Level AA`
> - **Preprocessors & Frameworks**: Dart Sass (`sass`) `1.80+`, PostCSS `8.4+`, Tailwind CSS `4.0+` / `3.4+`
> - **Static Analyzers**: Stylelint `16.x` (`stylelint-config-standard`, `stylelint-order`), HTMLHint / ESLint JSX-A11y / Angular Template Analyzer, `prettier-plugin-tailwindcss` `0.6+`

---

## 1. HTML & Component Markup Style Rules (JSX / TSX / Angular / Vue / Svelte)

Every HTML template and component markup file must satisfy these structural, semantic, and security invariants:

| # | Rule Mandate | Rationale & Defect Prevented |
| :--- | :--- | :--- |
| **HT-01** | **Lowercase Element & Attribute Names** (`<button disabled>`) | Enforces standard HTML5 serialization (except framework-required PascalCase component tags like `<PatientCard />` or SVG camelCase `viewBox`). |
| **HT-02** | **Double Quotes (`"..."`) for All Attribute Values** | Never use unquoted attributes (`class=btn`) or single quotes in HTML/JSX attributes (`className="card-header"`). |
| **HT-03** | **Boolean Attributes Without Redundant Values** | Write `<input required disabled />` instead of `<input required="required" disabled="true" />` (except ARIA state attributes which require explicit strings: `aria-expanded="true"` or `aria-hidden="true"`). |
| **HT-04** | **Mandatory Explicit `type` on Every `<button>`** | An untyped `<button>` inside a `<form>` defaults to `type="submit"`, triggering accidental form submissions and page reloads! Always declare `type="button"`, `type="submit"`, or `type="reset"` (`button-has-type`). |
| **HT-05** | **Mandatory `alt`, `width`, and `height` on Every `<img>`** | Informative images require descriptive `alt="..."`; purely decorative images require explicit empty `alt=""` (`role="presentation"`). Explicit `width` and `height` (or CSS `aspect-ratio`) reserve layout space and prevent Cumulative Layout Shift (`CLS`). |
| **HT-06** | **Mandatory `rel="noopener noreferrer"` on `target="_blank"` Links** | Prevents reverse tabnabbing (`window.opener` access) and referrer leakage when opening external domains. |
| **HT-07** | **Ban Inline `style="..."` Attributes for Static Styling** | Inline styles bypass CSS cascade layers, cannot respond to media queries or `:hover`/`:focus-visible`, and violate strict CSP `style-src`. **Sole Exception**: Passing dynamic runtime coordinates or progress values via CSS Custom Properties (`style={{ '--progress-ratio': ratio }}`). |
| **HT-08** | **Ban Positive `tabindex` Values (`tabindex="1"`, `tabindex="2"`)** | Positive `tabindex` hijacks natural DOM keyboard tab order (`WCAG 2.4.3`). Only `tabindex="0"` (add custom element to natural tab sequence) and `tabindex="-1"` (programmatic focus target) are allowed. |
| **HT-09** | **Ban Click Handlers on Non-Interactive Elements (`<div onClick>`)** | Attaching click handlers to `<div>`, `<span>`, or `<li>` breaks keyboard navigation (`Enter`/`Space`), screen reader role announcement, and focus rings. Always use semantic `<button type="button">` or `<a href="...">`. |
| **HT-10** | **Explicit `<label>` Association for Every Form Control** | Every `<input>`, `<select>`, and `<textarea>` must be paired with an explicit `<label for="id">` (or `aria-labelledby`) and link validation errors via `aria-describedby` and `aria-invalid="true"`. |
| **HT-11** | **Unique IDs Across the Document Tree** | Never hardcode static `id="email-input"` inside a reusable component that may render multiple times on a page (breaking `<label for>` and `aria-describedby`). Generate scoped IDs (`useId()` in React/Vue/Angular). |

```html
<!-- ❌ ANTI-PATTERN: Missing button type + clickable div + positive tabindex + static inline style -->
<div class="modal-Card" style="margin-top: 20px; color: #333333">
  <div onclick="closeModal()" tabindex="2">X</div>
  <img src="/clinic-map.webp" />
  <button onclick="saveDraft()">Save Draft</button>
</div>

<!-- ✅ STAFF PATTERN: Semantic elements + explicit button types + accessible attributes + zero inline styles -->
<article class="modal-card" aria-labelledby="modal-title-clinic">
  <button type="button" class="modal-card__close-btn" aria-label="Close clinic map dialog">
    <svg aria-hidden="true" viewBox="0 0 24 24" width="20" height="20"><!-- ... --></svg>
  </button>
  <img
    src="/clinic-map.webp"
    alt="Street map showing Urgent Care Clinic entrance on 4th Avenue"
    width="640"
    height="360"
    loading="lazy"
  />
  <button type="button" class="btn btn--secondary">Save Draft</button>
</article>
```

---

## 2. CSS, SCSS & LESS Style & Architecture Rules

### 2.1 Selector Discipline & Specificity Ceilings
1. **Class Naming Convention**:
   - Use lowercase `kebab-case` (`.patient-summary-card`) or structured **BEM** (`.patient-card__header--highlighted`) when authoring vanilla CSS/SCSS/LESS outside CSS Modules.
   - In CSS Modules (`*.module.css`), map classes cleanly so HTML output remains inspectable in DevTools.
2. **Ban `#id` Selectors in Stylesheets (`selector-max-id: 0`)**:
   - An `#id` selector has specificity `1,0,0`, trumping any number of class selectors (`0,1,0`) and forcing escalating `!important` hacks. Reserve `id` attributes strictly for HTML `<label for>`, ARIA relationships, and URL hash anchors.
3. **Ban `!important` in Feature & Component Styles (`declaration-no-important`)**:
   - Using `!important` is a symptom of broken cascade architecture. Use CSS Cascade Layers (`@layer reset, tokens, base, components, utilities;`) to control precedence deterministically.
4. **Specificity & Nesting Budgets**:
   - **Maximum Compound Selectors**: `<= 3` (`.card .card__header .badge` max; never `.page div.wrapper ul li a.link`).
   - **Maximum Nesting Depth**: `<= 3` levels in native CSS Nesting or SCSS/LESS (`max-nesting-depth: 3`).
   - **Maximum Specificity**: `0,3,0`.
   - **Ban BEM Ampersand String Concatenation (`&__element`)**: In SCSS, writing `&__header` inside `.patient-card` makes searching the codebase for `.patient-card__header` with `grep` / `ripgrep` impossible! Only use `&` for pseudo-classes (`&:hover`, `&:focus-visible`), attribute states (`&[data-state='open']`), or modifier classes.

### 2.2 Deterministic 6-Group CSS Property Ordering
Within any CSS rule block, declare properties in this concentric **outside-in** sequence (enforced via `stylelint-order`):

| Group | Category | Properties Included (In Order) |
| :--- | :--- | :--- |
| **1. Positioning & Layering** | Where the box sits in the document/stack | `position`, `inset`, `inset-block`, `inset-inline` (`top`, `right`, `bottom`, `left`), `z-index`, `isolation` |
| **2. Display & Box Layout** | How the box participates in layout & arranges children | `display`, `flex`, `flex-direction`, `flex-wrap`, `justify-content`, `align-items`, `grid-template-columns`, `grid-template-rows`, `gap`, `order` |
| **3. Box Sizing & Logical Spacing** | Dimensions, overflow, padding, and margins | `box-sizing`, `inline-size` / `width`, `min-inline-size` / `min-width`, `max-inline-size` / `max-width`, `block-size` / `height`, `aspect-ratio`, `padding`, `padding-block`, `padding-inline`, `margin`, `margin-block`, `margin-inline`, `overflow` |
| **4. Typography & Text** | Font rendering, alignment, and wrapping | `font-family`, `font-size`, `font-weight`, `font-variant-numeric`, `line-height`, `letter-spacing`, `text-align`, `text-wrap`, `white-space`, `color` |
| **5. Visual Decoration & Surface** | Backgrounds, borders, shadows, opacity, cursor | `background`, `background-color`, `border`, `border-radius`, `outline`, `box-shadow`, `opacity`, `visibility`, `cursor`, `pointer-events` |
| **6. Transitions & Animations** | Motion & state interpolation | `transition`, `transition-property`, `transition-duration`, `transition-timing-function`, `transform`, `animation` |

```css
/* ✅ STAFF PATTERN: concentric 6-group property order + logical properties + design tokens */
.settlement-summary-card {
  /* 1. Positioning */
  position: relative;
  z-index: var(--z-card);

  /* 2. Display & Layout */
  display: flex;
  flex-direction: column;
  gap: var(--space-4);

  /* 3. Sizing & Logical Spacing */
  inline-size: 100%;
  max-inline-size: 42rem;
  padding-block: var(--space-6);
  padding-inline: var(--space-5);

  /* 4. Typography */
  font-family: var(--font-sans);
  font-size: var(--text-base);
  font-variant-numeric: tabular-nums;
  line-height: 1.5;
  color: var(--color-text-primary);

  /* 5. Visual Surface */
  background-color: var(--color-surface-elevated);
  border: 1px solid var(--color-border-subtle);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);

  /* 6. Motion */
  transition:
    transform 180ms var(--ease-out),
    box-shadow 180ms var(--ease-out);
}
```

### 2.3 Units, Logical Properties & Design Tokens
- **Relative Units (`rem`) for Typography & Spacing**: Never hardcode `font-size: 14px`, which breaks when a visually impaired user scales their browser default font size from `16px` to `24px` (`WCAG 1.4.4`). Use `rem` (`0.875rem`, `1rem`, `1.25rem`) for font sizes, spacing, and media query breakpoints.
- **`px` Permitted Only for Hairline Borders & Shadows**: `1px` or `2px` borders and focus ring offsets (`outline-offset: 2px`) should remain in `px`.
- **Unitless `0` and Unitless `line-height`**: Always write `margin: 0;` (never `0px` or `0rem`) and unitless `line-height: 1.5;` (never `line-height: 24px`, which fails to scale proportionally in child elements).
- **Logical Properties Over Physical Directions**: Prefer `margin-inline-start`, `padding-inline`, `inset-inline-end`, `border-inline-start`, and `text-align: start` over physical `left` and `right` so layouts support bidirectional (`LTR` / `RTL`) localization automatically.
- **Zero Raw Hex/RGB Literals in Components**: Feature stylesheets must reference semantic CSS custom properties (`var(--color-surface-primary)`, `var(--color-status-danger)`). Raw color coordinates (`oklch(...)`, `hsl(...)`, `#hex`) belong exclusively inside the central `:root` / `@theme` token definition file.
- **Modern CSS Color Function Syntax**: When defining tokens, use modern space-separated syntax with slash alpha: `oklch(0.62 0.19 255 / 0.15)` or `rgb(15 23 42 / 0.8)` (ban legacy `rgba(15, 23, 42, 0.8)`).

### 2.4 SCSS / SASS / LESS Preprocessor Rules
1. **Ban `@import`; Enforce `@use` and `@forward` (Dart Sass `1.80+`)**:
   - Sass `@import` pollutes the global namespace, executes files multiple times, and is officially deprecated/removed in modern Dart Sass. Always use `@use './tokens' as t;` with explicit namespacing.
2. **Partials Emit Zero Raw CSS on Import**:
   - Shared `_variables.scss` and `_mixins.scss` partials must contain **only** compile-time declarations (`$breakpoints`, `@mixin`, `@function`). Importing a partial must never duplicate compiled CSS rules across every component bundle.
3. **Prefer Native CSS Custom Properties (`--var`) Over Preprocessor `$vars` for Theme Values**:
   - Use SCSS `$variables` only for compile-time media query breakpoints (where CSS `var()` cannot be used inside `@media (min-width: ...)`). Use native CSS `--custom-properties` for colors, spacing, typography, and shadows so dark mode and runtime theming require zero CSS duplication.

---

## 3. Tailwind CSS Utility Style Rules (`Tailwind v4.0 / v3.4`)

1. **Deterministic Utility Class Ordering**:
   - Enforce automatic class sorting (`prettier-plugin-tailwindcss` or Biome `useSortedClasses`) following the canonical box-model order:
     `Layout/Position -> Flex/Grid -> Sizing -> Spacing (p/m) -> Typography -> Backgrounds/Borders/Effects -> Transitions -> State Variants (hover:/focus-visible:) -> Responsive/Theme Variants (sm:/md:/dark:)`.
2. **Strictly Ban Dynamic Class String Interpolation**:
   - **Never** write ``className={`bg-${statusColor}-500 text-${statusColor}-900`}``. Tailwind's static AST scanner cannot detect partial class fragments at build time, causing the styles to be purged in production! Always map state keys to **complete static utility strings**:
     ```tsx
     // ❌ ANTI-PATTERN: Dynamic string interpolation (purged in production build!)
     const badgeClass = `bg-${status}-100 text-${status}-800`;

     // ✅ STAFF PATTERN: Complete static string lookup map + exhaustive Record type
     const STATUS_BADGE_CLASSES: Record<OrderStatus, string> = {
       pending: 'bg-amber-100 text-amber-900 border-amber-300',
       settled: 'bg-emerald-100 text-emerald-900 border-emerald-300',
       disputed: 'bg-rose-100 text-rose-900 border-rose-300',
     };
     ```
3. **Ban `@apply` Abuse in Stylesheets**:
   - Do not recreate traditional BEM stylesheets filled with `@apply flex items-center px-4 py-2 ...`. Extract reusable UI primitives (`<Button variant="primary">`, `<Badge status="settled">`) at the component level (using `cva` / `class-variance-authority` or `tailwind-variants`).
4. **Ban Arbitrary Value Escapes (`w-[347px]`, `text-[#1a2b3c]`) When Tokens Exist**:
   - Arbitrary bracket values (`[...]`) are an escape hatch for one-off geometry, never a substitute for registering semantic design tokens in `@theme` / `tailwind.config`.
