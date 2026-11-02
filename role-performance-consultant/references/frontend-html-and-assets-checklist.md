# Frontend Performance Checklist: HTML Structure, Resource Hints & Media Assets

> **Baseline Specifications & Tooling Versions**:
> - **Markup & Navigation**: HTML Living Standard (WHATWG 2026), Speculation Rules API (WICG / Chrome 121+), Fetch Priority API
> - **Media & Vector Formats**: AVIF (`image/avif`), WebP (`image/webp`), SVG 2.0 + SVGO 3.3+, AV1 / VP9 (`video/webm`)
> - **Core Web Vitals Targets**: LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1 (75th Percentile Mobile)

---

## 1. DOM Size, Depth, and Layout Complexity Budgets

- **Rule**: Keep total document DOM nodes **< 1,500**, maximum tree depth **< 32 levels**, and maximum child nodes per parent **< 60** (unless virtualized).
- **Why**: Large DOM trees increase memory footprint, slow down style recalculation ($O(N)$ per style mutation), and cause prolonged layout/reflow phases during user interactions (degrading INP).
- **Multi-Domain Application**:
  - *E-commerce*: Do not render 500 product cards or hidden mega-menu sub-trees in the initial DOM; render sub-menus on demand or use `<template>`.
  - *Healthcare*: Do not mount hidden modal dialogs for every row in a 200-patient ward roster; share a single singleton `<dialog>` instance populated dynamically.
  - *FinTech*: Do not keep 2,000 historical order-book ticks mounted in hidden tabs; unmount inactive tab panels or virtualize the viewport.

---

## 2. Early Network Connections (`preconnect` & `dns-prefetch`)

- **Rule**: Use `<link rel="preconnect" href="https://..." crossorigin>` strictly for critical origins required within the first 2 seconds (max 2–4 origins). Use `<link rel="dns-prefetch">` as a lightweight fallback for secondary third-party origins.
- **Anti-Pattern**: Preconnecting to 10+ origins (analytics, chat widgets, social embeds), which exhausts browser socket pools and contends for CPU during TLS handshakes on low-end mobile devices.

```html
<!-- GOOD: Preconnect only to critical CDN and API origins -->
<link rel="preconnect" href="https://cdn.example-platform.org" crossorigin />
<link rel="preconnect" href="https://api.example-platform.org" crossorigin />
<!-- Fallback DNS lookup for deferred analytics/maps -->
<link rel="dns-prefetch" href="https://tiles.maps-provider.org" />
```

---

## 3. Resource Preloading & Module Preloading (`preload` & `modulepreload`)

- **Rule**: Use `<link rel="preload">` only for late-discovered critical resources needed for initial paint (e.g., primary WOFF2 font, CSS-referenced hero background image). Use `<link rel="modulepreload">` for critical ES module chunks to flatten dependency waterfalls.
- **Pitfall**: Preloading a font without `crossorigin="anonymous"` causes a **double download** (once anonymous via preload, once CORS-enabled via `@font-face`).

```html
<!-- GOOD: Font preload with explicit type and crossorigin -->
<link
  rel="preload"
  href="/fonts/inter-var-latin.woff2"
  as="font"
  type="font/woff2"
  crossorigin="anonymous"
/>
<link rel="modulepreload" href="/assets/app-core.a1b2c3.js" />
```

---

## 4. Priority Hints (`fetchpriority="high" | "low"`)

- **Rule**: Boost the LCP element (e.g., E-commerce hero banner, Civic election live map header, News lead photo) with `fetchpriority="high"`. Downgrade non-critical above-the-fold secondary images (e.g., carousel slides 2–5, footer badges) with `fetchpriority="low"`.

---

## 5. Script Execution Scheduling (`async`, `defer`, `type="module"`)

- **Rule**: Never put synchronous `<script src="...">` tags in `<head>` without `defer` or `async`.
  - `defer` (and `<script type="module">` by default): Downloads in parallel without blocking HTML parsing; executes in order after DOM parsing completes. Use for application bundles.
  - `async`: Downloads in parallel and executes *immediately* upon download completion, interrupting HTML parsing or rendering. Use only for independent, lightweight scripts that do not touch the DOM synchronously.

---

## 6. High-Performance Image Markup (`<img>`, `<picture>`, AVIF/WebP, `srcset`, `sizes`)

- **Rule**:
  1. **LCP Hero Image**: Must **never** have `loading="lazy"`. Must have `fetchpriority="high"`, explicit `width` and `height` attributes (to reserve aspect ratio and prevent CLS), and responsive `srcset` + `sizes`.
  2. **Below-the-Fold Images**: Must have `loading="lazy"` and `decoding="async"` so image rasterization occurs off the main thread.
  3. **Modern Formats**: Serve AVIF (`image/avif`) with WebP (`image/webp`) and JPEG/PNG fallbacks via `<picture>`.

```html
<!-- Hero LCP Image (e.g., E-commerce flagship product / Real estate listing hero) -->
<picture>
  <source
    type="image/avif"
    srcset="/img/hero-400.avif 400w, /img/hero-800.avif 800w, /img/hero-1200.avif 1200w"
    sizes="(max-width: 768px) 100vw, 50vw"
  />
  <source
    type="image/webp"
    srcset="/img/hero-400.webp 400w, /img/hero-800.webp 800w, /img/hero-1200.webp 1200w"
    sizes="(max-width: 768px) 100vw, 50vw"
  />
  <img
    src="/img/hero-800.jpg"
    width="1200"
    height="800"
    fetchpriority="high"
    alt="Primary visual showcase"
  />
</picture>

<!-- Below-the-fold thumbnail (e.g., Healthcare doctor avatar / Civic candidate portrait) -->
<img
  src="/img/avatar-160.webp"
  width="80"
  height="80"
  loading="lazy"
  decoding="async"
  alt="Profile portrait"
/>
```

---

## 7. SVG Optimization & Sprite Sheet Architecture

- **Rule 1 (SVGO Build Pipeline)**: Run all SVG assets through **SVGO** (`svgo.config.mjs`) to strip editor metadata (`<metadata>`, `xmlns:xlink`, Sketch/Figma comments), collapse useless groups, convert shapes to paths, and round numeric coordinates to 2 decimal places (typically 40%–70% byte reduction).
- **Rule 2 (Avoid Inline SVG DOM Node Inflation)**: Never inline complex multi-path `<svg>` icons repeatedly inside large lists or tables (e.g., 3 action icons per row across 200 rows in a SaaS CRM or FinTech ledger = 600 `<svg>` elements and 2,400+ `<path>` nodes).
  - **Why**: Every inline `<svg>` and `<path>` is a full DOM element that participates in style recalculation, hit-testing, and memory allocation.
  - **Best Practice**: Reference a single cached SVG sprite sheet via `<svg><use href="/assets/icons.svg#icon-id"></use></svg>` or render purely decorative single-color icons via CSS `mask-image` / background sprites.

```html
<!-- ANTI-PATTERN: Repeating 15-node inline <svg> inside 200 table rows (3,000+ extra DOM nodes) -->

<!-- BEST PRACTICE: Referencing an external browser-cached SVG sprite (2 DOM nodes per icon) -->
<button type="button" class="icon-btn" aria-label="Download statement">
  <svg width="20" height="20" aria-hidden="true" focusable="false">
    <use href="/assets/icons.v4.svg#icon-download"></use>
  </svg>
</button>
```

---

## 8. Video & Third-Party Iframe Facade Patterns

- **Rule 1 (Never Use Animated GIFs)**: Replace all animated GIFs with muted, looping `<video>` elements encoded in **AV1 / VP9 (`.webm`)** and **H.264 (`.mp4`)**, reducing file sizes by **80%–95%**.
- **Rule 2 (Non-Autoplaying Video Preload)**: For instructional or product videos that do not autoplay, always set `preload="none"` and provide an optimized `poster="/img/poster.webp"` attribute so the browser downloads **0 bytes** of video segments until the user clicks play.
- **Rule 3 (Third-Party Iframe Facades / Click-to-Load)**: Never mount raw third-party `<iframe>` embeds (YouTube, Vimeo, Google Maps, interactive 3D tours, customer chat widgets) directly during initial page load.
  - **Why**: A single YouTube or interactive map iframe pulls in 800 KB–1.5 MB of uncacheable third-party JS, fires 25+ network requests, and blocks the main thread for 400ms+.
  - **Facade Pattern**: Render a lightweight static HTML facade (`<button>` with a responsive `<picture>` thumbnail and play/map icon). Warm up connections with `<link rel="preconnect">` on `pointerenter`, and swap the facade for the real `<iframe allow="autoplay">` only on `click`.

```html
<!-- GOOD: Lightweight Video / Map Facade (0 KB third-party JS on initial load) -->
<div
  class="video-facade"
  data-embed-src="https://www.youtube-nocookie.com/embed/dQw4w9WgXcQ?autoplay=1"
>
  <picture>
    <source srcset="/img/clinic-tour-poster.avif" type="image/avif" />
    <img
      src="/img/clinic-tour-poster.webp"
      width="1280"
      height="720"
      loading="lazy"
      decoding="async"
      alt="Virtual tour of the outpatient cardiology clinic"
    />
  </picture>
  <button type="button" class="facade-play-trigger" aria-label="Play virtual tour video">
    <svg width="64" height="64" aria-hidden="true"><use href="/assets/icons.svg#play"></use></svg>
  </button>
</div>
```

---

## 9. Table Layout & Inert `<template>` Markup

- **Rule (`table-layout: fixed`)**: Always apply `table-layout: fixed` to large data tables (e.g., FinTech order books, SaaS audit logs, Healthcare lab panels) with explicit `<col>` widths. Without `table-layout: fixed`, the browser cannot render the first row until it has parsed the entire `<table>` and calculated the widest cell in every column.
- **Rule (`<template>`)**: Use native `<template>` tags for inert HTML fragments that are stamped out dynamically via JS. Content inside `<template>` is parsed into an inert `DocumentFragment` without downloading images, executing scripts, or inflating the active layout tree.

---

## 10. Speculative Navigation (`<script type="speculationrules">`)

- **Rule**: Use the standard **Speculation Rules API** to prefetch or prerender high-probability next navigations (e.g., E-commerce product detail links from a category grid, Civic precinct detail pages, SaaS dashboard sections) on hover (`"eagerness": "moderate"`) or pointer down (`"eagerness": "conservative"`).
- **Safety Guardrails**:
  1. **Explicitly exclude** state-mutating or session-sensitive URLs (`/logout`, `/api/*`, `/cart/add`, `/checkout/*`) using `{ "not": { "href_matches": [...] } }`.
  2. Use `"prefetch"` for broad link sets and reserve `"prerender"` for high-confidence targets or elements opted-in via `data-prerender` attributes to respect browser memory limits (Chrome caps concurrent prerenders).

```html
<script type="speculationrules">
  {
    "prefetch": [
      {
        "where": {
          "and": [
            { "href_matches": "/*" },
            { "not": { "href_matches": ["/logout", "/auth/*", "/checkout/*", "/api/*"] } },
            { "not": { "selector_matches": "[rel~=nofollow], [data-no-prefetch]" } }
          ]
        },
        "eagerness": "moderate"
      }
    ],
    "prerender": [
      {
        "where": { "selector_matches": "a[data-prerender='true']" },
        "eagerness": "conservative"
      }
    ]
  }
</script>
```

---

## 11. HTML Streaming & Early `<head>` Flush Boundaries

- **Rule**: In Server-Side Rendered (SSR) applications, structure the HTTP response to flush the opening `<!DOCTYPE html><html><head>...</head>` chunk **immediately** (`Transfer-Encoding: chunked`) before awaiting slow backend database or microservice calls for the `<body>`.
- **Why**: Flushing `<head>` in the first 20–50ms allows the browser's speculative HTML preload scanner to immediately parse `<link rel="preconnect">`, `<link rel="preload">`, critical CSS, and `<script type="module" src="...">` and begin downloading them over the network while the server is still querying the database for the page body.
