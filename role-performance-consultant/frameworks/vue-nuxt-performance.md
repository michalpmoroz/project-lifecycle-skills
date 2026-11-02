# Vue 3.5+ & Nuxt 3.14+ Performance Guide

> **Baseline Framework & Tooling Versions**:
> - **Vue Core**: Vue 3.5+ (Refactored reactivity system, `shallowRef`, `v-memo`, `v-once`, `defineAsyncComponent` Lazy Hydration strategies, Vapor-ready patterns)
> - **Meta-Framework**: Nuxt 3.14+ (Nitro Server Engine, Nuxt Islands `.server.vue`, Hybrid `routeRules`, `useFetch` / `useAsyncData` caching)
> - **Core Web Vitals Targets**: LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1

---

## 1. Avoiding Deep Reactivity Overhead (`shallowRef` & `shallowReactive`)

- **Rule**: Vue's default `ref()` and `reactive()` recursively wrap every nested object and array element in ES6 `Proxy` instances. When storing large, read-heavy datasets returned from an API (e.g., 5,000 GeoJSON precinct polygons, 10,000 product SKUs, Healthcare lab histories, or ECharts datasets), **always use `shallowRef()`**.
- **Why**: `shallowRef()` only tracks `.value` reassignment at the root reference, avoiding the allocation of tens of thousands of nested reactivity proxies and cutting memory/CPU initialization cost by 80%+.

```ts
import { shallowRef } from 'vue';

// GOOD: Zero deep Proxy allocation for 10,000 immutable records
const auditLogEntries = shallowRef<ReadonlyArray<AuditEntry>>([]);

async function loadEntries() {
  const data = await fetch('/api/v1/audit-logs').then((r) => r.json());
  // Reassigning .value triggers reactivity cleanly
  auditLogEntries.value = Object.freeze(data.items);
}
```

---

## 2. Subtree Memoization (`v-memo` & `v-once`)

- **Rule**:
  - Use `v-once` on purely static subtrees that never change after initial mount.
  - Use `v-memo="[item.id, item.selected, item.status]"` on complex `v-for` rows so Vue skips virtual DOM diffing completely for rows whose listed dependency values have not changed.

```vue
<template>
  <div
    v-for="row in rows"
    :key="row.id"
    v-memo="[row.id === selectedRowId, row.updatedAt]"
    class="data-row"
  >
    <ComplexRowContent :row="row" :is-selected="row.id === selectedRowId" />
  </div>
</template>
```

---

## 3. Computed Stability (Vue 3.4 / 3.5+)

- **Rule**: Return primitive values or stable references from `computed()`. If a `computed` returns a new object literal every time, downstream watchers or child components will re-run unnecessarily.
- **Pattern**: In Vue 3.4+, `computed` only triggers dependents if the computed value actually changed (`Object.is`). Derive primitive booleans (`const isRowSelected = computed(() => selectedId.value === props.id)`) rather than passing a changing `selectedId` prop down into 500 child row components.

---

## 4. Bounding `<KeepAlive :max="N">` Cache to Prevent Memory Leaks

- **Rule**: Whenever wrapping dynamic components or `<RouterView>` in `<KeepAlive>`, **always set an explicit `:max="N"` LRU bound** (e.g., `:max="5"`) and use `include` / `exclude` to restrict caching strictly to lightweight stateful views.
- **Why**: An unbounded `<KeepAlive>` never unmounts visited routes or tabs—keeping their entire DOM subtrees, reactive watchers, and memory buffers alive in RAM indefinitely as the user navigates across 50+ pages (a major cause of mobile tab crashes in SaaS, EHR, and E-commerce portals).
- **Lifecycle Hygiene**: Always pause timers, polling loops, or heavy canvas animations inside `onDeactivated()` and resume them in `onActivated()`.

```vue
<template>
  <RouterView v-slot="{ Component }">
    <!-- Bound LRU cache to at most 5 recently visited views -->
    <KeepAlive :max="5" :include="['CatalogSearchView', 'PatientRosterView']">
      <component :is="Component" />
    </KeepAlive>
  </RouterView>
</template>
```

---

## 5. Lazy Hydration (`hydrateOnVisible` / `hydrateOnInteraction`) & Nuxt Islands

- **Rule (Vue 3.5 Built-in Lazy Hydration)**: Use `defineAsyncComponent` with Vue 3.5's native hydration strategies (`hydrateOnVisible()`, `hydrateOnIdle()`, `hydrateOnInteraction('click')`) so below-the-fold SSR components do not block main-thread hydration.
- **Rule (Nuxt 3.14+ Hybrid Route Rules & Server Islands)**:
  - Configure `routeRules` in `nuxt.config.ts` to apply `swr` (Stale-While-Revalidate) or `prerender: true` on public content routes (e.g., E-commerce category pages, Civic candidate profiles, Healthcare clinic directories).
  - Use `.server.vue` Nuxt Island components for complex static sections (e.g., markdown documentation, legal footers, complex SVG charts) to ship **0 KB of client-side JavaScript** for those subtrees.

```ts
import { defineAsyncComponent, hydrateOnVisible } from 'vue';

const LazyAnalyticsChart = defineAsyncComponent({
  loader: () => import('./components/AnalyticsChart.vue'),
  hydrate: hydrateOnVisible({ rootMargin: '150px' }),
});
```

---

## 6. Nuxt Data Fetching: `pick`, `transform` & Client-Side Cache Hits (`getCachedData`)

- **Rule 1 (Payload Pruning with `pick` / `transform`)**: Everything returned from `useFetch()` or `useAsyncData()` during SSR is serialized into the HTML document inside `window.__NUXT__`. Always use `pick: ['id', 'title', 'priceCents']` or `transform` to strip unused fields before serialization.
- **Rule 2 (Instant Client Cache Hits with `getCachedData`)**: By default, Nuxt re-fetches `useFetch` data on client-side route navigation even if the data was already fetched 10 seconds ago. Provide a `getCachedData(key, nuxtApp)` callback to serve valid cached payloads directly from `nuxtApp.payload.data[key]` with **0ms network latency**.

```ts
// composables/useCatalogCategories.ts
export function useCatalogCategories() {
  return useFetch('/api/v1/categories', {
    key: 'catalog-categories',
    // Strip bloated backend fields from SSR payload serialization
    transform: (response) =>
      response.items.map((c) => ({ id: c.id, slug: c.slug, name: c.name })),
    // Serve from in-memory Nuxt payload cache on client-side back/forward navigation
    getCachedData(key, nuxtApp) {
      return nuxtApp.payload.data[key] || nuxtApp.static.data[key];
    },
  });
}
```

---

## 7. Vue 3.5 & Nuxt 3.14 Quick Audit Matrix

| Bottleneck / Anti-Pattern | Vue 3.5+ / Nuxt 3.14+ Remediation |
| :--- | :--- |
| **Deep Proxy Overhead on Large Arrays** | Store large API lists/GeoJSON in `shallowRef()` + `Object.freeze()`. |
| **Expensive List Re-Renders** | Add `:key="item.id"` and `v-memo="[item.id === selectedId, item.updatedAt]"`. |
| **Unbounded `<KeepAlive>` RAM Leak** | Always set `<KeepAlive :max="5" :include="[...]">` and clean up in `onDeactivated()`. |
| **Hydration Main-Thread Bloat** | Use `defineAsyncComponent({ hydrate: hydrateOnVisible() })` and `.server.vue` Islands. |
| **Bloated `window.__NUXT__` SSR Payload** | Use `pick` or `transform` in `useFetch` / `useAsyncData`. |
| **Redundant Client-Side Re-Fetching** | Configure `getCachedData(key, nuxtApp)` in `useFetch` / `useAsyncData`. |
| **LCP Image Regression** | Use `<NuxtImg src="..." width="..." height="..." preload fetchpriority="high" format="avif,webp" />`. |
