# Frontend Performance Checklist: Build Tooling, Bundling, Compression & Asset Delivery

> **Baseline Specifications & Tooling Versions**:
> - **Bundlers & Compilers**: Vite 6.x (Rollup 4 / Rolldown), esbuild 0.24+, SWC 1.10+, Rspack 1.2+ / Webpack 5.97+, Lightning CSS 1.28+
> - **Target Environments**: ECMAScript 2022+ (`es2022` / Baseline Widely Available), WebAssembly 2.0 (`WebAssembly.instantiateStreaming`)
> - **Compression Algorithms**: Brotli (`br` quality 11), Zstandard (`zstd` level 19), Gzip (`gzip -9` fallback)

---

## 1. Tree-Shaking & The Barrel File (`index.ts`) Anti-Pattern

- **Rule**:
  1. Declare `"sideEffects": false` (or `"sideEffects": ["**/*.css"]`) in `package.json` so bundlers (Vite/Rollup, Webpack, Rspack, esbuild) can safely drop unused exports.
  2. **Eliminate deep Barrel Files (`index.ts` that re-export `export * from './...'` across 50 modules)**: Importing a single helper from a massive barrel file forces bundler module graphs and test runners (Vitest/Jest) to parse and evaluate hundreds of unrelated modules, bloating chunks and slowing cold builds by 50%+.
  3. Import modularly (e.g., prefer native `structuredClone`, `Array.prototype.at`, `Object.groupBy` over importing full `lodash`; prefer `date-fns` individual functions or `Temporal` polyfills over `moment`).

```json
// package.json — Enable aggressive bundler tree-shaking while preserving CSS imports
{
  "name": "@org/ui-components",
  "sideEffects": [
    "**/*.css",
    "**/*.scss"
  ]
}
```

---

## 2. Code Splitting & Dynamic `import()` Architecture

- **Rule**: Split bundles along three axes:
  1. **Route-level splitting**: Every page route loads its own lazy chunk.
  2. **Below-the-fold / Conditional UI splitting**: Heavy interactive modals, rich-text editors (TipTap/ProseMirror), charting libraries (ECharts/Chart.js), PDF viewers (PDF.js), and map SDKs (MapLibre/Leaflet) must be loaded lazily via `await import('./heavy-chart')` only when the user opens the tab or scrolls near the container.
  3. **Vendor chunk stability**: Separate stable framework core chunks (`framework`, `vendor-ui`) from rapidly changing feature code so user browsers retain cached vendor bundles across deployments.

---

## 3. Modern ES Target Bundling (`target: 'es2022'`) & Eliminating Polyfill Bloat

- **Rule**: Configure TypeScript (`tsconfig.json`) and bundler build targets (`vite.config.ts` / `esbuild` / `swc`) to **`es2022` (or `baseline-widely-available`)** instead of legacy `es5` or `es2015`.
- **Why**:
  1. **Downlevel Transpilation Tax**: Compiling native `async/await`, classes, private fields (`#prop`), optional chaining (`?.`), and nullish coalescing (`??`) down to ES5/ES2015 state-machine generators inflates JavaScript bundle size by **15%–30%** and executes significantly slower in V8/JavaScriptCore than native syntax.
  2. **Redundant `core-js` Polyfills**: Never blindly import `core-js/stable` or polyfill features supported by 99%+ of evergreen browsers (`Promise`, `fetch`, `URL`, `Array.prototype.includes`, `Object.entries`, `Intl`, `structuredClone`).

```typescript
// vite.config.ts — Modern ES2022 target + Lightning CSS + deterministic vendor splitting
import { defineConfig } from 'vite';

export default defineConfig({
  build: {
    target: 'es2022',
    cssMinify: 'lightningcss',
    sourcemap: 'hidden',
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-core': ['zod', 'date-fns'],
        },
      },
    },
  },
});
```

---

## 4. CSS Code-Splitting, Lightning CSS & Unused CSS Purging

- **Rule 1 (Per-Route CSS Code-Splitting)**: Keep `cssCodeSplit: true` enabled in your bundler so each lazy-loaded route chunk only downloads the CSS rules required for that route, rather than blocking the homepage on a monolithic 300 KB stylesheet covering admin dashboards and settings modals.
- **Rule 2 (Unused CSS Purging)**: Use Tailwind v4 / v3 content scanning or PurgeCSS to strip unused utility and component classes at build time. Ensure dynamic class names are always written as complete static strings in source code (`isActive ? 'bg-emerald-600' : 'bg-slate-200'`, never `'bg-' + color + '-600'`).
- **Rule 3 (Lightning CSS Minification)**: Prefer **Lightning CSS** over legacy CSS minifiers to deduplicate shorthand properties, transpile modern CSS nesting/color-spaces (`oklch()`), and shrink CSS payloads by an additional 5%–12%.

---

## 5. Minification, Dead-Code Elimination & Obfuscation Performance Warnings

- **Minification & Dead-Code Elimination**: Use SWC, esbuild, or Terser for AST minification (`drop_console: ['log', 'debug']`, constant folding, dead branch elimination via compile-time flags like `import.meta.env.PROD`).
- **Obfuscation Performance Warning**: **Avoid aggressive JavaScript obfuscation** (control-flow flattening, string-array encoding, dead-code injection, `eval` wrapping) on production web bundles.
  - **Why**: Obfuscation increases bundle size by **30%–200%**, destroys Brotli compression dictionary efficiency, and defeats V8/JSC JIT compiler optimizations (preventing function inlining and hidden-class prediction), resulting in **2x–10x slower runtime execution** and severe mobile battery drain. Never rely on client-side JS obfuscation for security—keep sensitive algorithms, proprietary scoring rules, and secrets strictly on the backend.

---

## 6. WebAssembly (`.wasm`) Streaming Compilation (`WebAssembly.instantiateStreaming`)

- **Rule**: When using WebAssembly modules (e.g., client-side image compression with Squoosh/libvips, SQLite-in-WASM for offline healthcare records, CAD/GeoJSON geometry engines, or cryptographic verification), **always instantiate via `WebAssembly.instantiateStreaming(fetch('/module.wasm'), imports)`**.
- **Server Header Requirement**: The origin server or CDN **must** serve `.wasm` files with `Content-Type: application/wasm` (compressed with Brotli `br`).
- **Why**: `instantiateStreaming` compiles WebAssembly bytecode into native machine code **in parallel while the network bytes are still streaming in**, cutting WASM initialization latency by 50%–80% compared to fetching the full `ArrayBuffer` first and calling `WebAssembly.instantiate(buffer)`.

```typescript
// ANTI-PATTERN: Waiting for full download + buffering in RAM before compiling WASM
// const bytes = await fetch('/assets/geo-engine.wasm').then((r) => r.arrayBuffer());
// const { instance } = await WebAssembly.instantiate(bytes, importObject);

// BEST PRACTICE: Streaming compilation directly from the HTTP Response stream
export async function loadWasmEngine(importObject: WebAssembly.Imports = {}): Promise<WebAssembly.Instance> {
  const responsePromise = fetch('/assets/geo-engine.a1b2c3.wasm');
  const { instance } = await WebAssembly.instantiateStreaming(responsePromise, importObject);
  return instance;
}
```

---

## 7. Static Pre-Compression: Brotli (`br` q11) & Zstandard (`zstd` level 19)

- **Rule**: Pre-compress immutable static assets (`.js`, `.css`, `.svg`, `.wasm`, `.json`, `.ttf`) during the CI/CD build step using **Brotli at quality level 11 (`br -q 11`)** and **Zstandard (`zstd -19`)**, alongside Gzip (`gzip -9`) fallback.
- **Why**: Dynamic on-the-fly compression at the web server/reverse proxy typically uses Brotli level 4 or Gzip level 6 to save CPU. Build-time pre-compression at maximum dictionary levels (`br -q 11` / `zstd -19`) yields **15%–25% smaller payloads** with zero runtime CPU cost on the origin server.

---

## 8. Production Source Maps Strategy

- **Rule**: Never inline source maps (`inline-source-map`) or expose public `//# sourceMappingURL=` comments if proprietary source size or IP is a concern, and ensure source map files (`.js.map`) are never downloaded by regular users. Upload `.map` files directly to error-monitoring artifacts (e.g., self-hosted GlitchTip/Sentry) during CI and configure the bundler with `sourcemap: 'hidden'`.
