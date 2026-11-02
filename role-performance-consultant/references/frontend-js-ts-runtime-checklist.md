# Frontend Performance Checklist: JavaScript/TypeScript Runtime, Memory, Workers & UI Architecture

> **Baseline Specifications & Tooling Versions**:
> - **Language & Runtimes**: ECMAScript 2025 (ES16), TypeScript 5.7+, V8 12.x / JavaScriptCore / SpiderMonkey
> - **Browser Web APIs**: Prioritized Task Scheduling API (`scheduler.postTask`, `scheduler.yield`), Long Animation Frames (LoAF API), Web Workers, `SharedWorker`, `BroadcastChannel`, Web Locks API, Web Crypto API (`crypto.subtle`), Constructable Stylesheets (`CSSStyleSheet`)
> - **Core Web Vitals Targets**: INP ≤ 200ms, Zero Long Tasks (> 50ms) on Main Thread, Zero Memory Leaks over 24h SPA sessions

---

## 1. Breaking Up Long Tasks (> 50ms) & Improving INP (`scheduler.yield()`)

- **Rule**: Any synchronous JavaScript execution exceeding 50ms is classified as a **Long Task** and blocks the browser from painting the next frame or responding to user input.
- **Pattern**: In event handlers (e.g., clicking "Apply Filter" in E-commerce, "Submit Claim" in Healthcare, or "Sort District Results" in Civic apps), update the critical UI feedback first, then yield to the main thread using `scheduler.yield()` (with fallback to `setTimeout(resolve, 0)`) before running expensive data processing or telemetry.

```typescript
async function yieldToMain(): Promise<void> {
  if ('scheduler' in globalThis && 'yield' in (globalThis as any).scheduler) {
    return (globalThis as any).scheduler.yield();
  }
  return new Promise((resolve) => setTimeout(resolve, 0));
}

export async function handleFilterClick(items: ReadonlyArray<RecordItem>): Promise<void> {
  // 1. Immediate visual acknowledgment so browser can paint next frame (< 16ms)
  setFilterButtonLoadingState(true);

  // 2. Yield to let the compositor paint the loading state (fixes INP!)
  await yieldToMain();

  // 3. Process chunks without blocking main thread for > 50ms at a time
  const results: RecordItem[] = [];
  const CHUNK_SIZE = 500;
  for (let i = 0; i < items.length; i += CHUNK_SIZE) {
    const slice = items.slice(i, i + CHUNK_SIZE);
    results.push(...slice.filter(matchesComplexCriteria));
    if (i + CHUNK_SIZE < items.length) {
      await yieldToMain();
    }
  }

  renderFilteredResults(results);
}
```

---

## 2. Task Prioritization: `requestAnimationFrame` vs. `requestIdleCallback` vs. `scheduler.postTask()`

- **Rule**: Schedule main-thread work according to its urgency using the right browser scheduling primitive:

| API | Priority Tier | When to Use | Anti-Pattern |
| :--- | :--- | :--- | :--- |
| `requestAnimationFrame(cb)` | **Pre-Paint Frame** (Runs right before Style/Layout/Paint) | Synchronizing visual DOM writes, FLIP geometry measurements, custom canvas drawing. | Running analytics, API fetches, or state serialization inside `rAF` (causes frame drops!). |
| `scheduler.postTask(cb, { priority: 'user-blocking' })` | **Highest** (Input response) | Immediate UI state feedback to a click/keypress. | Scheduling background prefetching as `user-blocking`. |
| `scheduler.postTask(cb, { priority: 'user-visible' })` | **Medium** (Default) | Rendering secondary non-critical UI widgets or charts. | Blocking input handlers. |
| `scheduler.postTask(cb, { priority: 'background' })` / `requestIdleCallback(cb)` | **Lowest** (Idle periods only) | Sending analytics beacons, warming route caches, indexing local storage. | Mutating visible DOM styles inside `requestIdleCallback` (causes unpredictable layout recalculation). |

```typescript
export function scheduleBackgroundTelemetry(payload: Record<string, unknown>, signal?: AbortSignal): void {
  if ('scheduler' in globalThis && 'postTask' in (globalThis as any).scheduler) {
    (globalThis as any).scheduler.postTask(() => sendAnalytics(payload), {
      priority: 'background',
      signal,
    });
  } else if ('requestIdleCallback' in globalThis) {
    requestIdleCallback(() => sendAnalytics(payload), { timeout: 3000 });
  } else {
    setTimeout(() => sendAnalytics(payload), 200);
  }
}
```

---

## 3. Eliminating Forced Synchronous Layout (Layout Thrashing)

- **Rule**: Never interleave DOM geometry reads (`element.offsetHeight`, `getBoundingClientRect()`, `getComputedStyle()`, `scrollTop`) with DOM style/class writes inside a loop.
- **Why**: Reading geometry after mutating a style forces the browser to abort JS execution and run a synchronous reflow immediately on the main thread.

```typescript
// ANTI-PATTERN: Forced Synchronous Layout (O(N^2) layout thrashing)
for (const box of boxes) {
  const width = box.offsetWidth; // Forces synchronous layout on EVERY iteration!
  box.style.height = `${width * 0.75}px`; // Invalidates layout!
}

// BEST PRACTICE: Batch all reads first, then batch all writes (or use CSS aspect-ratio!)
const widths = boxes.map((box) => box.offsetWidth); // Single layout pass
boxes.forEach((box, index) => {
  box.style.height = `${widths[index] * 0.75}px`;
});
```

---

## 4. Offloading Heavy Compute to Web Workers, `OffscreenCanvas` & Zero-Copy Transferables

- **Rule 1 (Web Workers & `OffscreenCanvas`)**: Move CPU-intensive operations off the main UI thread into dedicated Web Workers:
  - Client-side fuzzy search / indexing (e.g., searching 20,000 medical ICD-10 codes or product SKUs).
  - Cryptographic hashing, CSV/Excel parsing, GeoJSON polygon triangulation, or PDF rendering.
  - Complex 2D/WebGL data visualizations via `OffscreenCanvas`.
- **Rule 2 (Avoid Structured Clone Overhead on Large Objects)**:
  - Calling `structuredClone(largeState)` or `worker.postMessage(largeObject)` serializes and deep-copies the entire object graph, blocking the main thread for 50ms–300ms when payloads exceed several megabytes.
  - **Remedies**:
    1. Use **Transferable Objects** (`worker.postMessage(arrayBuffer, [arrayBuffer])`) to transfer ownership of binary buffers (`ArrayBuffer`, `MessagePort`, `ImageBitmap`, `OffscreenCanvas`) across threads in **$O(1)$ zero-copy time (< 0.01ms)**.
    2. In main-thread state management, use **structural sharing** (updating only modified object branches via immutable spread or Immer) instead of deep-cloning untouched subtrees.

---

## 5. Cross-Tab Resource Deduplication (`BroadcastChannel` & `SharedWorker`)

- **Rule**: Power users in FinTech (trading terminals), Healthcare (EHR multi-patient tabs), Civic monitoring (live election dashboards), and SaaS often keep **5 to 20 browser tabs open** to the same application. Never open a separate WebSocket/SSE connection and run duplicate polling timers in every tab.
- **Pattern**: Use a **`SharedWorker`** (or **`navigator.locks` + `BroadcastChannel`** leader election) so **only 1 active leader tab/worker** maintains the remote WebSocket/SSE stream and broadcasts incoming events to all open follower tabs over an in-memory `BroadcastChannel` at zero network cost.

```typescript
// Leader-Election WebSocket Multiplexing via Web Locks API + BroadcastChannel
const syncChannel = new BroadcastChannel('live-market-feed');

export function initMultiplexedFeed(onMessage: (data: unknown) => void): () => void {
  const listener = (event: MessageEvent) => onMessage(event.data);
  syncChannel.addEventListener('message', listener);

  // Only one tab acquires the exclusive lock to hold the physical WebSocket connection
  if ('locks' in navigator) {
    navigator.locks.request('live-market-feed-leader', async () => {
      await new Promise<void>((resolve) => {
        const ws = new WebSocket('wss://stream.example-platform.org/v1/ticks');
        ws.onmessage = (e) => {
          const parsed = JSON.parse(e.data);
          onMessage(parsed); // Deliver to leader tab
          syncChannel.postMessage(parsed); // Broadcast to all follower tabs
        };
        ws.onclose = () => resolve(); // Release lock if socket closes or tab unloads
      });
    });
  }

  return () => syncChannel.removeEventListener('message', listener);
}
```

---

## 6. Memory Leak Prevention & Garbage Collection Hygiene

Audit long-lived Single Page Applications (SPAs) against the 5 classic memory leak vectors:
1. **Detached DOM Trees**: Storing references to `HTMLElement` nodes in global caches, closures, or module-level variables after those nodes have been removed from the `document`. Use `WeakRef<HTMLElement>` or `WeakMap<HTMLElement, Metadata>` whenever associating state with DOM nodes.
2. **Dangling Event Listeners & AbortController**: Every `window.addEventListener`, `document.addEventListener`, or long-lived bus listener must be cleaned up on teardown. Prefer passing an `AbortSignal` from an `AbortController` to remove multiple listeners in one call (`controller.abort()`).
3. **Uncleaned Timers & Observers**: Always clear `setInterval`, `requestAnimationFrame`, `ResizeObserver.disconnect()`, `IntersectionObserver.disconnect()`, and `MutationObserver.disconnect()` when components unmount.
4. **Unbounded Closures & Module Caches**: Never push to module-scoped arrays/maps without an eviction policy (LRU cache with max entries or `WeakMap`).
5. **Forgotten Observable/Stream Subscriptions**: Ensure every RxJS `Observable`, WebSocket stream, or EventSource is unsubscribed on component destruction.

```typescript
// GOOD: Associating metadata with DOM nodes without preventing Garbage Collection
const elementMetadata = new WeakMap<HTMLElement, { initializedAt: number; chartId: string }>();

// GOOD: One-shot listener cleanup via AbortController
const lifecycleController = new AbortController();
window.addEventListener('resize', onResize, { passive: true, signal: lifecycleController.signal });
window.addEventListener('scroll', onScroll, { passive: true, signal: lifecycleController.signal });
// On component destroy:
lifecycleController.abort();
```

---

## 7. V8 Engine & JIT Optimization (Hidden Classes / Shapes & Monomorphism)

- **Rule**: Help modern JS JIT compilers (V8, JavaScriptCore, SpiderMonkey) maintain **monomorphic inline caches (ICs)** on hot code paths (e.g., parsing 100,000 financial ticks or geospatial coordinates):
  1. Initialize all object properties in the same order inside constructors or factory functions (assign `null` or `undefined` rather than adding properties conditionally later).
  2. **Never use the `delete` operator** on hot objects (`delete obj.prop` transitions the object into slow dictionary/hash-map mode in V8). Set `obj.prop = undefined` instead, or use `Map<K, V>` for dynamic key-value collections.
  3. Avoid mixing element kinds in hot numeric arrays (e.g., inserting `undefined`, `NaN`, or strings into a packed integer array transitions `PACKED_SMI_ELEMENTS` to slow `PACKED_ELEMENTS` or `HOLEY_ELEMENTS`). Use `Float64Array` / `Int32Array` (`TypedArrays`) for numeric buffers.

---

## 8. Passive Event Listeners & Observer APIs vs. Scroll Polling

- **Rule**:
  - Always attach `touchstart`, `touchmove`, `wheel`, and `scroll` listeners with `{ passive: true }`. Without `passive: true`, the browser compositor thread must wait for the main JS thread to finish executing the listener (in case it calls `event.preventDefault()`) before scrolling the page, causing severe scroll jank.
  - Replace `scroll` event polling (`getBoundingClientRect()`) with `IntersectionObserver` for visibility/infinite-scroll triggers and `ResizeObserver` for element dimension changes.

---

## 9. Virtual Scrolling / Windowing & Infinite Scroll Pitfalls

- **Rule**: Any list, grid, or table capable of exceeding **100 rows** (e.g., FinTech transaction ledgers, Healthcare lab result logs, E-commerce catalog grids, Civic voter precinct directories) must use **DOM Virtualization / Windowing** (rendering only the ~15–25 visible items plus a small overscan buffer) or paginated slices.
- **Infinite Scroll Pitfalls**:
  1. **Unbounded DOM Growth**: Naive infinite scroll that only appends nodes without unmounting offscreen top nodes eventually hits 10,000+ DOM nodes and crashes mobile Safari/Chrome.
  2. **Scroll Anchoring Thrashing**: Use `overflow-anchor: none` on custom virtual scroll containers where JavaScript manually calculates scroll offsets, preventing browser scroll-anchoring from fighting JS positioning.
  3. **Footer Inaccessibility**: Never place critical links (Privacy Policy, Terms, Support) below an infinite-scrolling feed without a sticky utility bar or manual "Load More" pagination option.

---

## 10. Web Components & Shadow DOM Performance (`adoptedStyleSheets`)

- **Rule**: When building Web Components (`customElements`), never inject a `<style>` tag inside every instance's Shadow Root (which forces the browser to parse and allocate duplicate `CSSStyleSheet` objects for 1,000 component instances).
- **Best Practice**: Construct a single shared `CSSStyleSheet` once and attach it to each Shadow Root via `shadowRoot.adoptedStyleSheets = [sharedSheet]`.

```typescript
const sharedBadgeSheet = new CSSStyleSheet();
sharedBadgeSheet.replaceSync(`
  :host { display: inline-flex; contain: content; }
  .label { font-variant-numeric: tabular-nums; }
`);

class StatusBadge extends HTMLElement {
  constructor() {
    super();
    const root = this.attachShadow({ mode: 'open' });
    root.adoptedStyleSheets = [sharedBadgeSheet]; // Zero duplicate style parsing!
  }
}
```

---

## 11. Internationalization (i18n) & `Intl` Formatter Caching

- **Rule 1 (Lazy Locale Dictionaries)**: Never bundle all language translation JSON files into the main JS entry bundle. Dynamically `import()` only the active locale dictionary at runtime (or resolve translations on the server).
- **Rule 2 (`Intl` Instantiation Cost)**: Instantiating `new Intl.DateTimeFormat()`, `new Intl.NumberFormat()`, or `new Intl.RelativeTimeFormat()` is **extremely expensive** (reads ICU locale tables). Never instantiate `Intl` formatters inside a render function or array `.map()` loop. Cache formatter instances in a module-level `Map` keyed by `${locale}-${optionsHash}`.

```typescript
// ANTI-PATTERN: Instantiating Intl.NumberFormat 1,000 times inside a table render loop (~120ms CPU!)
function renderPriceBad(amount: number, locale: string, currency: string): string {
  return new Intl.NumberFormat(locale, { style: 'currency', currency }).format(amount);
}

// BEST PRACTICE: Singleton cached formatter lookup (< 0.5ms for 1,000 calls!)
const numberFormatterCache = new Map<string, Intl.NumberFormat>();

export function formatCurrency(amount: number, locale: string, currency: string): string {
  const cacheKey = `${locale}:${currency}`;
  let formatter = numberFormatterCache.get(cacheKey);
  if (!formatter) {
    formatter = new Intl.NumberFormat(locale, { style: 'currency', currency });
    numberFormatterCache.set(cacheKey, formatter);
  }
  return formatter.format(amount);
}
```

---

## 12. Accessibility + Performance Synergy

- **Rule (`prefers-reduced-motion`)**: Honor `@media (prefers-reduced-motion: reduce)` by disabling decorative parallax, auto-playing video loops, and complex particle canvases. This simultaneously satisfies WCAG 2.2 AA (SC 2.3.3) and saves significant CPU/GPU battery on mobile devices.
- **Rule (AOM Inflation)**: Avoid adding redundant `aria-*` attributes or `role` declarations to thousands of purely decorative wrapper `<div>` elements, as this bloats the browser's Accessibility Object Model (AOM) serialization during screen-reader tree updates.

---

## 13. Security vs. Performance Trade-Offs

- **Web Crypto API (`crypto.subtle`)**: Never use pure-JS implementations of SHA-256, AES, or PBKDF2 on the main thread. Always use native browser `globalThis.crypto.subtle` (which runs in compiled C++/Rust threads and uses hardware AES-NI instructions—10x to 100x faster and zero JS bundle weight).
- **Subresource Integrity (SRI) + Preload**: When using `integrity="sha384-..."` on third-party scripts/styles, ensure CORS headers match so the browser does not discard preloaded assets.
- **CSP Overhead**: Prefer clean, concise Content Security Policy headers using cryptographic nonces or hashes over 20KB+ domain allowlists transmitted on every HTTP response header.
