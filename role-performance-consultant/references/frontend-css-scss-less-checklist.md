# Frontend Performance Checklist: CSS, SCSS/SASS/LESS, Layout Containment & Web Fonts

> **Baseline Specifications & Tooling Versions**:
> - **CSS Specifications**: CSS Snapshot 2025/2026 (CSS Containment Level 3, View Transitions Level 2, CSS Cascade Layers `@layer`, CSS Transitions Level 2 `@starting-style` & `allow-discrete`)
> - **Preprocessors & Postprocessors**: Dart Sass 1.80+, Less 4.2+, PostCSS 8.4+, Lightning CSS 1.28+
> - **Core Web Vitals Targets**: LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1 (75th Percentile Mobile)

---

## 1. Critical CSS Inlining & Non-Blocking Stylesheets

- **Rule**: Extract and inline above-the-fold Critical CSS (< 14 KB compressed to fit within the first TCP congestion window) directly into `<head><style>...</style></head>`. Load the rest of the stylesheet asynchronously or via modular route chunks.
- **Anti-Pattern**: Using CSS `@import url("...")` inside stylesheets. `@import` forces sequential network waterfalls instead of parallel downloads.

---

## 2. Offscreen Rendering Bypass (`content-visibility: auto`)

- **Rule**: Apply `content-visibility: auto` paired with `contain-intrinsic-size` to major below-the-fold page sections (e.g., E-commerce customer reviews, Healthcare patient history sections, SaaS documentation footers).
- **Impact**: Instructs the browser to skip layout, style calculation, and painting for offscreen subtrees until they approach the viewport—often reducing initial rendering work by 40%–70%.

```css
.below-fold-section {
  content-visibility: auto;
  /* Reserves placeholder height (width: auto, height: 600px) to prevent scrollbar jumping */
  contain-intrinsic-size: auto 600px;
}
```

---

## 3. CSS Containment (`contain: layout style paint`)

- **Rule**: Isolate independent widgets (e.g., live stock tickers in FinTech, real-time vote counters in Civic apps, notification toasts) using `contain: layout style paint` (or `contain: content`).
- **Why**: Guarantees to the browser engine that internal DOM/style changes inside the widget cannot affect the geometry of any ancestor or sibling outside the widget, bounding reflows to a tiny subtree.

---

## 4. Compositor-Only Animations vs. Layout/Paint Thrashing

- **Rule**: Animate **only** properties handled directly by the GPU compositor thread: `transform` (`translate`, `scale`, `rotate`), `opacity`, and `filter` (with caution).
- **Anti-Pattern**: Animating `top`, `left`, `width`, `height`, `margin`, `padding`, or `border-width`, which triggers main-thread **Layout (Reflow) → Paint → Composite** on every single frame (dropping below 60fps/120fps and blocking input handlers).
- **Pitfall (`will-change` abuse)**: Never apply `will-change: transform, opacity` globally (e.g., `* { will-change: transform; }` or on 100 list items). Each promoted compositor layer consumes dedicated GPU VRAM texture memory, causing mobile browsers to crash or experience severe layer-tree thrashing. Apply `will-change` dynamically only right before an animation starts and remove it afterward.

---

## 5. GPU Memory Traps: `backdrop-filter` & Complex `box-shadow`

- **Rule**: Minimize simultaneous `backdrop-filter: blur(...)` surfaces and large-radius animated `box-shadow` declarations on mobile viewports.
- **Why**: `backdrop-filter` requires the GPU to sample, copy, and run a Gaussian convolution shader over the underlying framebuffer every frame the background scrolls. For hover/elevation shadows, animate `opacity` on a pseudo-element (`::after`) that holds a pre-painted static `box-shadow`.

```css
/* GOOD: Zero-repaint shadow transition via compositor opacity */
.card {
  position: relative;
}
.card::after {
  content: "";
  position: absolute;
  inset: 0;
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.18);
  opacity: 0;
  transition: opacity 200ms ease-out;
  pointer-events: none;
}
.card:hover::after {
  opacity: 1;
}
```

---

## 6. Native CSS Entry/Exit Transitions (`@starting-style` & `allow-discrete`) vs. JS Animation Libraries

- **Rule**: For modal dialogs (`<dialog>`), popovers (`[popover]`), dropdowns, and toast notifications entering or leaving the DOM (`display: none` ↔ `display: block`), use native CSS `@starting-style` and `transition-behavior: allow-discrete` instead of importing 30 KB+ JavaScript animation libraries or scheduling double-`requestAnimationFrame` class-toggling hacks.
- **Why**: Runs 100% on the browser's native compositor thread with zero JS main-thread orchestration overhead and zero bundle size cost.

```css
/* Native zero-JS entry and exit transition for top-layer <dialog> or [popover] */
dialog[open] {
  opacity: 1;
  transform: scale(1);
}

dialog {
  opacity: 0;
  transform: scale(0.96);
  transition:
    opacity 180ms ease-out,
    transform 180ms ease-out,
    overlay 180ms allow-discrete,
    display 180ms allow-discrete;
}

@starting-style {
  dialog[open] {
    opacity: 0;
    transform: scale(0.96);
  }
}
```

---

## 7. CSS View Transitions API Performance (`startViewTransition` & `@view-transition`)

- **Rule**: Use the native View Transitions API (`document.startViewTransition()` or cross-document `@view-transition { navigation: auto; }`) for smooth state and route morphing (e.g., morphing an E-commerce product thumbnail into the detail hero image, or transitioning between Healthcare patient chart tabs).
- **Performance Guardrails**:
  1. **Limit Active `view-transition-name` Layers**: Never assign unique `view-transition-name` values to 50+ list items simultaneously. Every named element requires the browser to capture a separate GPU snapshot texture before the transition starts, causing a noticeable frame freeze (`> 100ms` INP penalty) on mobile devices. Assign `view-transition-name` dynamically only to the 1–3 elements actively participating in the transition.
  2. **Respect Reduced Motion**: Disable or collapse morph geometry animations inside `@media (prefers-reduced-motion: reduce)`.

```css
@view-transition {
  navigation: auto;
}

@media (prefers-reduced-motion: reduce) {
  ::view-transition-group(*),
  ::view-transition-old(*),
  ::view-transition-new(*) {
    animation-duration: 0.01ms !important;
  }
}
```

---

## 8. Selector Matching Efficiency & Avoiding `:has()` / Substring Attribute Explosions

- **Rule**: Browsers evaluate CSS selectors **from right to left** (starting at the rightmost "key selector"). Keep key selectors specific (single class name) and avoid expensive universal or unanchored relational selectors in large DOM trees:
  1. **Avoid Substring Attribute Selectors on Hot Paths**: Rules like `[class*="col-"]`, `[data-id^="row-"]`, or `[href*="http"]` require string scanning on every candidate element during style recalculation. Use explicit classes (`.col`, `.external-link`).
  2. **Constrain `:has()` Scope**: Never write unanchored or deep `:has(*)` selectors like `.page:has(.input:invalid)` or `div:has(> .active)` when thousands of `div` elements exist. Anchor `:has()` to a narrow parent container class (`.form-field:has(> .input:invalid)`) so style invalidation only checks immediate children of `.form-field`.

---

## 9. CSS Cascade Layers (`@layer`) vs. Specificity Bloat

- **Rule**: Declare an explicit `@layer` order at the top of your stylesheet (`@layer reset, base, tokens, components, utilities;`) to control cascade precedence deterministically.
- **Why**: Eliminates the need for high-specificity selector chaining (`.app-shell .sidebar .nav-item .btn.btn-active`) or `!important` overrides. `@layer` resolution is computed during initial CSS cascade sorting with **zero runtime layout/paint penalty**, keeping selectors flat (single-class key selectors) and fast to match.

```css
@layer reset, base, tokens, components, utilities;

@layer components {
  .btn {
    padding-inline: 1rem;
  }
}

@layer utilities {
  /* Wins over .btn deterministically without increasing selector specificity! */
  .px-0 {
    padding-inline: 0;
  }
}
```

---

## 10. SCSS / SASS / LESS Preprocessor Pitfalls

- **Selector Nesting Explosion**:
  - **Rule**: Never nest SCSS/LESS selectors deeper than 3 levels. Avoid parent selector (`&`) cartesian product explosions inside loops.
  - **Why**: Deeply nested rules like `.dashboard .panel .table tr td a span` force the browser engine to walk up the DOM tree for every `<span>` on the page.
- **`@mixin` vs. `%placeholder` (`@extend`) Bloat**:
  - **Pitfall**: Including a 40-line `@mixin` inside 50 component selectors duplicates those 40 lines 50 times in the compiled CSS output (though gzip mitigates repetition, CSS parse time and CSSOM memory still scale linearly with uncompressed rule count). Use atomic utility classes or CSS custom properties instead of massive structural mixins.
- **Compile-Time Functions vs. Runtime CSS Custom Properties (`var(--*)`)**:
  - **Rule**: Use SCSS/LESS math and color functions (`color.scale()`, `math.div()`) at compile time for static constants. Use CSS Custom Properties (`--theme-surface`) for dynamic runtime theming.
  - **Pitfall**: Mutating a CSS Custom Property (`--scroll-y` or `--mouse-x`) on `:root` or `<html>` inside a `mousemove` or `scroll` handler invalidates styles for **every single element in the document**. Scope dynamic CSS variables strictly to the specific leaf element being animated.

---

## 11. Web Font Loading & CLS Zero-Shift Metrics

- **Rule**:
  1. Always serve **WOFF2** (`font-woff2`), preferably as a single **Variable Font** replacing multiple weight files (`400`, `500`, `600`, `700`).
  2. Subset fonts by script using `unicode-range` (e.g., splitting Latin, Latin-Ext, Cyrillic).
  3. Use `font-display: swap` (or `font-display: optional` for non-essential decorative fonts) paired with metric overrides (`size-adjust`, `ascent-override`, `descent-override`, `line-gap-override`) on the local fallback `@font-face` so swapping fonts causes **0.000 CLS**.

```css
@font-face {
  font-family: "Inter Fallback";
  src: local("Arial");
  size-adjust: 107.4%;
  ascent-override: 90.2%;
  descent-override: 22.4%;
  line-gap-override: 0%;
}

@font-face {
  font-family: "Inter";
  src: url("/fonts/inter-var-latin.woff2") format("woff2");
  font-weight: 100 900;
  font-display: swap;
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+2000-206F;
}

body {
  font-family: "Inter", "Inter Fallback", sans-serif;
}
```
