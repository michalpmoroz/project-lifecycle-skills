# Svelte 5 & SvelteKit 2 Performance Guide

> **Baseline Framework & Tooling Versions**:
> - **Svelte Core**: Svelte 5.x (Runes: `$state`, `$state.raw`, `$derived`, `$derived.by`, `$effect`, `untrack`)
> - **Meta-Framework**: SvelteKit 2.x (Streaming Server Load promises, Speculative link preloading `data-sveltekit-preload-data`, `@sveltejs/enhanced-img`)
> - **Core Web Vitals Targets**: LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1

---

## 1. Svelte 5 Runes: `$state.raw` vs. Deeply Proxied `$state`

- **Rule**: In Svelte 5, `$state({...})` creates a deep reactive `Proxy` for objects and arrays so fine-grained property mutations (`item.count += 1`) trigger targeted updates. However, for large read-only or replaced-as-a-whole arrays (e.g., 10,000 financial candle ticks, Healthcare lab result tables, E-commerce search results, or GeoJSON feature collections), **always use `$state.raw(...)`**.
- **Why**: `$state.raw` disables deep property instrumentation—mutating an inner property is a no-op, and updates happen strictly by reassigning the array/object reference (`items = newItems`), yielding maximum raw JS speed and zero `Proxy` allocation overhead.

```svelte
<script lang="ts">
  interface Transaction {
    id: string;
    amountCents: number;
    status: 'settled' | 'pending';
  }

  // GOOD: Zero deep proxy overhead for large API collections
  let transactions = $state.raw<ReadonlyArray<Transaction>>([]);
  let filterStatus = $state<'all' | 'settled' | 'pending'>('all');

  // Fine-grained derived state
  let visibleTransactions = $derived(
    filterStatus === 'all'
      ? transactions
      : transactions.filter((t) => t.status === filterStatus)
  );
</script>

<ul>
  {#each visibleTransactions as tx (tx.id)}
    <li>{tx.id}: {tx.amountCents / 100}</li>
  {/each}
</ul>
```

---

## 2. Eliminating `$effect` Over-Firing & Using `untrack()`

- **Rule 1 (`$derived` Over `$effect`)**: Never use `$effect(() => { derivedCount = items.length * 2; })` to synchronize state. Synchronizing state inside `$effect` causes an extra post-render microtask cycle and double DOM updates. Always use `$derived(...)` or `$derived.by(() => ...)`.
- **Rule 2 (Isolating Non-Trigger Reads with `untrack()`)**: `$effect` automatically tracks **every** `$state` or `$derived` signal read synchronously inside its execution body (including inside helper functions called by the effect). When an effect should only re-run when `primaryId` changes—not when secondary state like `userPreferences` or `analyticsSession` changes—wrap the secondary reads in `untrack(() => ...)`.

```svelte
<script lang="ts">
  import { untrack } from 'svelte';

  let selectedDistrictId = $state('dist-01');
  let currentZoomLevel = $state(12);

  $effect(() => {
    // Tracked dependency: effect re-runs ONLY when selectedDistrictId changes
    const id = selectedDistrictId;

    // Untracked read: zooming the map will NOT re-trigger fetchDistrictDetails!
    const zoom = untrack(() => currentZoomLevel);

    const controller = new AbortController();
    fetchDistrictDetails(id, zoom, controller.signal);

    // Teardown function automatically aborts previous request on re-run or unmount
    return () => controller.abort();
  });
</script>
```

---

## 3. Mandatory Keyed `{#each}` Blocks

- **Rule**: Always provide a unique, stable domain key expression in `{#each items as item (item.id)}`.
- **Pitfall**: Unkeyed `{#each items as item}` updates DOM nodes in place by array index when items are inserted, sorted, or deleted at the top/middle of the list, causing unnecessary DOM text/attribute mutations and subtle state bugs in child components.

---

## 4. SvelteKit Streaming Load Functions (`+page.server.ts`)

- **Rule**: In SvelteKit `+page.server.ts` / `+page.ts` `load` functions, `await` only the critical above-the-fold data required for SEO and LCP. Return slow secondary promises **un-awaited** as top-level properties of the returned object.
- **Why**: SvelteKit flushes and renders the initial HTML shell immediately once top-level `await` statements complete, and streams the resolved value of nested promises over the same HTTP response chunk stream into `{#await data.slowMetrics}` blocks without blocking Time to First Byte (TTFB) or LCP.

```ts
// src/routes/listings/[id]/+page.server.ts
import type { PageServerLoad } from './$types';

export const load: PageServerLoad = async ({ params }) => {
  // 1. Start slow secondary query immediately without awaiting yet
  const nearbyMetricsPromise = fetchNearbyDistrictMetrics(params.id);

  // 2. Await critical primary record for immediate SSR + LCP
  const primaryListing = await fetchListingById(params.id);

  return {
    listing: primaryListing,
    // 3. Return un-awaited Promise — SvelteKit streams it to the client!
    nearbyMetrics: nearbyMetricsPromise,
  };
};
```

---

## 5. SvelteKit Speculative Link Preloading (`data-sveltekit-preload-data`)

- **Rule**: Configure `data-sveltekit-preload-data="hover"` on the `<body>` or primary navigation containers so SvelteKit automatically prefetches both the target route's JavaScript bundle and its `load` function JSON data as soon as the user's pointer hovers over a link (giving a ~200ms head start before `click`).
- **Guardrails for Expensive / State-Mutating Links**:
  1. Downgrade links that trigger expensive backend database aggregations to `data-sveltekit-preload-data="tap"` (runs on `pointerdown`) or `data-sveltekit-preload-code="hover"` (preloads only the JS chunk, not the API data).
  2. Always set `data-sveltekit-preload-data="off"` on state-sensitive links (e.g., `/auth/logout`, `/export/pdf`).

```html
<!-- High-confidence navigation preloads data + code on hover -->
<nav data-sveltekit-preload-data="hover">
  <a href="/catalog/featured">Featured Catalog</a>
  <a href="/analytics/live-report" data-sveltekit-preload-data="tap">Heavy Report (Tap Only)</a>
  <a href="/auth/logout" data-sveltekit-preload-data="off">Sign Out</a>
</nav>
```

---

## 6. Svelte 5 & SvelteKit 2 Quick Audit Matrix

| Bottleneck / Anti-Pattern | Svelte 5 / SvelteKit 2 Remediation |
| :--- | :--- |
| **Deep Proxy Overhead on Large Lists** | Use `$state.raw<T[]>([])` instead of `$state<T[]>([])`. |
| **Cascading Double Renders from `$effect`** | Replace state-mutating `$effect` with `$derived` / `$derived.by`. |
| **Accidental `$effect` Re-Executions** | Wrap non-trigger signal reads inside `untrack(() => ...)`. |
| **DOM Thrashing on List Filter/Sort** | Always key loops: `{#each items as item (item.id)}`. |
| **Slow SSR TTFB from Secondary Data** | Return secondary Promises un-awaited from `+page.server.ts` and render with `{#await}`. |
| **Perceived Route Navigation Latency** | Use `data-sveltekit-preload-data="hover"` on nav links and `"tap"` / `"off"` on heavy/sensitive routes. |
| **LCP Image Regression** | Use `@sveltejs/enhanced-img` (`<enhanced:img src="..." fetchpriority="high" />`). |
