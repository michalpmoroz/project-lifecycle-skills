# Vue 3 & Nuxt 3 Senior/Staff Engineering Guide (`frameworks/vue.md`)

> **Baseline Versions**: Vue `3.5+`, Nuxt `3.14+`, TypeScript `5.7+`, Pinia `2.2+`, Vue Router `4.4+`, `vue-tsc` `2.1+`.
> **Mandate**: This guide establishes the mandatory baseline for modern Vue 3 and Nuxt 3 applications. Apply these idioms by default and continuously adopt newer stable Vue/Nuxt capabilities.

---

## 1. `<script setup lang="ts">` & Vue 3.5+ Component Macros

- **Composition API with `<script setup lang="ts">` Mandatory**: Never use the legacy Options API (`data()`, `methods`, `computed: {}`) or non-setup `setup()` functions in new components.
- **Component Block Order**: Keep `.vue` Single-File Components consistent: `<script setup lang="ts">` first, `<template>` second, `<style scoped>` third.

### 1.1 Vue 3.5+ Reactive Props Destructure
- In Vue 3.5+, destructured variables from `defineProps<T>()` are **natively reactive** (compiled to property accesses by the Vue compiler) and support standard JavaScript default values directly—eliminating the awkward `withDefaults()` macro!
- When passing a destructured prop into a composable or `watch`, wrap it in a getter `() => propName` to pass reactivity rather than its current value snapshot.

```vue
<script setup lang="ts">
import { computed, watch } from 'vue';

interface ProductCardProps {
  id: string;
  title: string;
  priceCents: number;
  currency?: 'USD' | 'EUR' | 'GBP';
  inStock?: boolean;
}

// Vue 3.5+ native reactive destructure with clean JS defaults
const {
  id,
  title,
  priceCents,
  currency = 'USD',
  inStock = true,
} = defineProps<ProductCardProps>();

// Reactive derived state tracks `priceCents` and `currency` automatically
const formattedPrice = computed(() =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency }).format(priceCents / 100)
);

// Watching a destructured prop requires a getter `() => id`
watch(() => id, (newId) => {
  // handle id transition
});
</script>
```

### 1.2 Succinct Typed `defineEmits`, `defineModel`, and `defineSlots`
- **`defineModel<T>()`**: Use `defineModel<string>({ required: true })` (or named `defineModel<boolean>('expanded', { default: false })`) for two-way binding instead of manually declaring `modelValue` props and `update:modelValue` emits.
- **`defineEmits<T>()`**: Use the succinct named-tuple syntax introduced in Vue 3.3+:

```ts
const emit = defineEmits<{
  selectCandidate: [candidateId: string, districtCode: string];
  clearFilters: [];
}>();
```

### 1.3 `useTemplateRef()` & `useId()` (Vue 3.5+)
- **`useTemplateRef`**: Always use `const dialogEl = useTemplateRef<HTMLDialogElement>('dialog')` instead of `const dialog = ref<HTMLDialogElement | null>(null)` to separate template DOM references clearly from reactive state refs.
- **`useId()`**: Always use `const inputId = useId()` for SSR-safe, hydration-stable `id` / `for` / `aria-describedby` bindings in reusable form controls.

---

## 2. Reactivity Primitives: `ref` vs. `shallowRef` vs. `reactive`

| Primitive | When to Use | Critical Pitfalls to Avoid |
| :--- | :--- | :--- |
| **`ref<T>()`** | **Default choice** for primitives, small objects, and local UI state. | Deeply wraps objects in `Proxy`. Avoid storing massive arrays (10,000+ rows) or third-party class instances in deep `ref()`. |
| **`shallowRef<T>()`** | **Mandatory** for large immutable datasets (e.g., GeoJSON civic maps, FinTech 50k-point price series, E-commerce catalog dumps) and external instances (Chart.js, MapLibre, Monaco Editor). | Only `.value = newValue` triggers updates; mutating nested properties (`data.value.items.push(...)`) will not trigger effects unless `triggerRef(data)` is called. |
| **`reactive<T>()`** | Optional for grouped local form state objects where every property is mutated together. | **Never destructure** a `reactive()` object directly (`const { x } = state` severs reactivity) and never reassign the root reference (`state = newState`). Prefer `ref()` for consistency. |

---

## 3. Composables Architecture (`use*` Conventions)

Composables (`src/composables/use*.ts`) are the primary mechanism for reusable stateful logic.

### 3.1 Flexible Input Acceptance (`MaybeRefOrGetter<T>` & `toValue()`)
- Accept arguments typed as `MaybeRefOrGetter<T>` so callers can pass a plain value, a `ref`, or a reactive getter `() => propVar`. Normalize inside the composable using `toValue(arg)`.

### 3.2 Automatic Cleanup (`onWatcherCleanup` & `onScopeDispose`)
- **Vue 3.5 `onWatcherCleanup()`**: Register abort controllers or timer cleanups directly inside `watch` or `watchEffect` callbacks before the first `await`.
- **`onScopeDispose()`**: Use `onScopeDispose(() => ...)` instead of `onUnmounted` inside composables so cleanup works inside both components and detached `effectScope()` contexts (like Pinia stores).

```ts
import { ref, watch, onWatcherCleanup, toValue, type MaybeRefOrGetter } from 'vue';

export function usePatientVitalsSearch(querySource: MaybeRefOrGetter<string>) {
  const results = shallowRef<PatientVitalSummary[]>([]);
  const isLoading = ref(false);
  const error = ref<string | null>(null);

  watch(
    () => toValue(querySource),
    async (query) => {
      if (!query.trim()) {
        results.value = [];
        return;
      }
      const controller = new AbortController();
      // Vue 3.5+ native watcher cleanup cancels in-flight fetch on next keystroke or unmount
      onWatcherCleanup(() => controller.abort());

      isLoading.value = true;
      error.value = null;
      try {
        const res = await fetch(`/api/vitals?q=${encodeURIComponent(query)}`, {
          signal: controller.signal,
        });
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        results.value = await res.json();
      } catch (err) {
        if ((err as Error).name !== 'AbortError') {
          error.value = 'Unable to load patient vitals.';
        }
      } finally {
        isLoading.value = false;
      }
    }
  );

  return { results, isLoading, error };
}
```

---

## 4. Pinia State Management & `storeToRefs`

- **Setup Stores Mandatory**: Always define Pinia stores using the **Setup Store syntax** (`defineStore('cart', () => { ... })`) rather than Option Stores. Setup stores offer full TypeScript inference and composable integration.
- **`storeToRefs()` Discipline**:
  - When destructuring state (`ref`) or getters (`computed`) from a Pinia store in a component, **always** wrap the store in `storeToRefs(store)`.
  - Actions (functions) can be destructured directly from `store`.

```ts
const cartStore = useCartStore();
// GOOD: Preserves reactivity on state and computed getters
const { items, totalAmount, isEmpty } = storeToRefs(cartStore);
// GOOD: Actions destructured directly from the store instance
const { addItem, removeItem, clearCart } = cartStore;
```

---

## 5. Nuxt 3 Data Fetching, Caching & SSR Safety

### 5.1 Cross-Request State Pollution (CRITICAL SSR SECURITY RULE)
> [!CAUTION]
> **Never declare mutable `ref()` or `reactive()` variables at module root scope outside a function/store in Nuxt SSR!**
> Module-level singletons persist across the Node.js/Nitro server process and will **leak one user's session, financial data, or medical records to another user's HTTP request**.
- Always use `useState('key', () => initialValue)` or Pinia stores (which are scoped to the active Nuxt `nuxtApp` request instance) for shared SSR-hydrated state.

### 5.2 `useFetch` & `useAsyncData` Best Practices
- **Never call raw `$fetch` in top-level `<script setup>`**: Calling raw `$fetch` during setup executes once on the server during SSR and *again* on the client during hydration (double-fetching).
  - Use **`useFetch` / `useAsyncData`** for initial component data loads (serializes payload from SSR to client without re-fetching).
  - Use **`$fetch`** exclusively inside user event handlers (`submitForm`, `loadMoreClick`).
- **Payload Minimizing (`pick` & `transform`)**: Always use `pick: ['id', 'title', 'price']` or `transform` in `useFetch` so Nuxt does not serialize unused 200KB API fields into the HTML `__NUXT_DATA__` payload.

```ts
const route = useRoute();
const { data: property, status, error, refresh } = await useFetch(
  () => `/api/properties/${route.params.id}`,
  {
    pick: ['id', 'title', 'price', 'coordinates', 'energyRating'],
    key: () => `property-${route.params.id}`,
  }
);
```

### 5.3 Preventing Hydration Mismatches
- Never render `Math.random()`, `new Date().toLocaleTimeString()`, or `localStorage` values directly in SSR templates.
- Use `<NuxtTime>` for locale-independent SSR time rendering, or wrap browser-only widgets in `<ClientOnly fallback-tag="div">` with a matching-height `#fallback` slot to prevent CLS.

---

## 6. Vue Router & Typed Navigation

- Use typed routes (`unplugin-vue-router` or Nuxt `experimental.typedPages: true`) so `RouterLink :to="{ name: '...', params: { ... } }"` is compile-time verified.
- Keep route guards (`router.beforeEach`) asynchronous, pure, and free of direct DOM manipulation; return route location objects (`return { name: 'login', query: { redirect: to.fullPath } }`) rather than calling legacy `next()`.

---

## 7. Vue 3 & Nuxt 3 Senior/Staff Checklist & Anti-Patterns

### Verification Checklist
- [ ] All components use `<script setup lang="ts">` with Vue 3.5+ reactive props destructuring, `defineModel`, `useTemplateRef`, and `useId`.
- [ ] Large read-only collections or 3rd-party class instances use `shallowRef` instead of deep `ref`.
- [ ] Pinia stores use Setup Store syntax, and consumers use `storeToRefs()` for state/getters.
- [ ] Zero module-scope mutable `ref()` singletons in SSR/Nuxt codebases.
- [ ] `v-for` directives always include a stable unique `:key="item.id"` and are never combined on the same element with `v-if` (use a `<template v-for>` wrapper or pre-filter via `computed()`).
- [ ] `vue-tsc --noEmit` passes with zero template or script type errors.

### Common Anti-Patterns to Reject Immediately in Code Review
1. **Mutating State Inside `computed()`**: Causes cascading side-effect loops and erratic UI updates.
2. **Destructuring Pinia Stores or `reactive()` Objects Without `storeToRefs`/`toRefs`**: Silently breaks reactivity so the UI freezes on its initial value.
3. **Combining `v-if` and `v-for` on the Same Element**: In Vue 3, `v-if` has higher precedence than `v-for` and cannot access the `v-for` loop variable. Filter the list in a `computed()` property first!
4. **Using Watchers to Compute Derived State**: Watching `firstName` and `lastName` to update `fullName.value = ...` instead of `const fullName = computed(...)`.
5. **Calling `useFetch` Inside Event Handlers**: `useFetch` is a setup-lifecycle composable; calling it inside `@click` handlers causes memory leaks and context errors. Use `$fetch` for imperative events.
