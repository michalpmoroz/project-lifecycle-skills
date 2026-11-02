# Svelte 5 & SvelteKit 2 Senior/Staff Engineering Guide (`frameworks/svelte.md`)

> **Baseline Versions**: Svelte `5.x` (Runes mode mandatory), SvelteKit `2.x`, TypeScript `5.7+`, `svelte-check` `4.x`.
> **Mandate**: This guide defines the mandatory baseline for modern Svelte 5 and SvelteKit 2 applications. All new code must use Svelte 5 Runes and Snippets; Svelte 3/4 legacy syntax (`export let`, `$:`, `<slot>`, `createEventDispatcher`) is strictly prohibited.

---

## 1. Svelte 5 Runes Mastery

| Svelte 3/4 Legacy (Prohibited) | Svelte 5 Rune (Mandatory) | Purpose |
| :--- | :--- | :--- |
| `let count = 0;` | `let count = $state(0);` | Deeply reactive mutable state |
| *(No direct equivalent)* | `let rows = $state.raw(largeData);` | Unproxied reference-level reactive state |
| `$: doubled = count * 2;` | `let doubled = $derived(count * 2);` | Synchronous derived state |
| `$: { complex block }` | `let total = $derived.by(() => { ... });` | Multi-statement derived calculation |
| `$: if (count) { sideEffect(); }` | `$effect(() => { ... });` | DOM/external side effects after render |
| `export let title = 'Default';` | `let { title = 'Default' }: Props = $props();` | Component input props |
| `export let value;` (two-way) | `let { value = $bindable('') }: Props = $props();` | Explicit two-way bindable prop |
| `<slot name="row" {item} />` | `{@render row(item)}` + `{#snippet row(item)}` | Type-safe composable template snippets |
| `on:click={handler}` | `onclick={handler}` | Standard HTML event attributes |
| `createEventDispatcher()` | Callback props (`onSave: (item: Item) => void`) | Strongly typed component events |

### 1.1 `$state`, `$state.raw`, and `$state.snapshot`
- **`$state(initial)`**: Creates a deeply reactive proxy for objects and arrays. Mutating `items.push(newItem)` or `user.profile.name = 'Alice'` automatically triggers fine-grained updates.
- **`$state.raw(initial)`**: Use for large read-only or immutable data structures (e.g., Civic GeoJSON election precincts, FinTech 100k order-book entries, Healthcare DICOM metadata arrays) where deep proxy wrapping adds unnecessary memory/CPU overhead. Updated only by reassigning the variable (`rows = nextRows`).
- **`$state.snapshot(stateProxy)`**: Unwraps a `$state` Proxy into a plain cloneable JavaScript object when passing state to `structuredClone`, `postMessage`, `IndexedDB`, or external non-Svelte libraries.

### 1.2 `$derived` and `$derived.by`
- Never use `$effect` to synchronize one `$state` variable from another. Always use `$derived(expression)` or `$derived.by(() => { ... })`.
- Note: In Svelte 5.25+, `$derived` values can also be temporarily overridden for optimistic UI patterns and automatically recompute when upstream dependencies change.

### 1.3 Strict `$effect` Discipline
- `$effect` runs on the client after the DOM has updated.
- Return a teardown function from `$effect(() => { ...; return () => cleanup(); })` to cancel timers, abort controllers, or third-party listeners before re-execution or component unmount.
- Use `untrack(() => ...)` when reading a reactive signal inside `$effect` that should **not** register as a re-run dependency.

---

## 2. Props, `$bindable`, Events & Snippets

### 2.1 Strictly Typed `$props()` and Callback Events
Never use `createEventDispatcher`. Pass typed callback props (`onSelect`, `onSubmit`) and extend native HTML attributes using `HTMLAttributes` or `HTMLButtonAttributes` from `'svelte/elements'`.

```svelte
<script lang="ts">
  import type { Snippet } from 'svelte';
  import type { HTMLAttributes } from 'svelte/elements';

  interface DataTableProps<T extends { id: string }> extends HTMLAttributes<HTMLTableElement> {
    items: readonly T[];
    selectedId?: string | null;
    onSelectRow?: (item: T) => void;
    headerSnippet: Snippet;
    rowSnippet: Snippet<[item: T, index: number]>;
    emptySnippet?: Snippet;
  }

  let {
    items,
    selectedId = $bindable(null),
    onSelectRow,
    headerSnippet,
    rowSnippet,
    emptySnippet,
    ...restProps
  }: DataTableProps<any> = $props();
</script>

<table {...restProps}>
  <thead>
    {@render headerSnippet()}
  </thead>
  <tbody>
    {#each items as item, index (item.id)}
      <tr
        aria-selected={selectedId === item.id}
        onclick={() => {
          selectedId = item.id;
          onSelectRow?.(item);
        }}
      >
        {@render rowSnippet(item, index)}
      </tr>
    {:else}
      <tr>
        <td colspan="100%">
          {#if emptySnippet}
            {@render emptySnippet()}
          {:else}
            No records match your active filter criteria.
          {/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>
```

---

## 3. Universal Reactivity (`.svelte.ts` Modules) & SSR Safety

- In Svelte 5, runes work outside `.svelte` components inside `.svelte.ts` files.
- **Reactive Classes**: Encapsulate domain logic (e.g., E-commerce cart calculator, Healthcare triage timer, SaaS undo/redo history stack) in classes with `$state` and `$derived` fields—Svelte 5 compiles class fields into ultra-fast V8 hidden classes!
- **CRITICAL SSR RULE — Never Export Global Mutable `$state` Singletons in SvelteKit**:
  - Exporting `export const userState = $state({ ... })` from a `.svelte.ts` module shares that state across **all concurrent HTTP requests** on the Node/Edge server.
  - Instead, instantiate the class inside `+layout.svelte` and distribute it down the component tree via Svelte's typed `setContext` / `getContext` API.

```ts
// src/lib/stores/triage-queue.svelte.ts
import { getContext, setContext } from 'svelte';

export class TriageQueueStore {
  patients = $state<PatientTriageItem[]>([]);
  criticalCount = $derived(this.patients.filter((p) => p.acuity === 'CRITICAL').length);

  addPatient(item: PatientTriageItem) {
    this.patients.push(item);
  }
}

const TRIAGE_KEY = Symbol('TRIAGE_QUEUE');

export function initTriageQueue(): TriageQueueStore {
  return setContext(TRIAGE_KEY, new TriageQueueStore());
}

export function useTriageQueue(): TriageQueueStore {
  return getContext<TriageQueueStore>(TRIAGE_KEY);
}
```

---

## 4. SvelteKit 2 Architecture: Load Functions & Form Actions

### 4.1 `+page.server.ts` vs. `+page.ts`
- Use `+page.server.ts` when accessing databases, private API keys, or filesystem resources.
- **SvelteKit 2 Error & Redirect Semantics**: In SvelteKit 2, `error(404, 'Not found')` and `redirect(303, '/login')` throw internally—you do not need `throw error(...)` (though calling them directly terminates execution immediately).
- **Streaming Non-Critical Promises**: Return top-level awaited promises for critical above-the-fold SEO data, and return **un-awaited** promises nested inside the return object for slow secondary widgets (e.g., E-commerce related items, Civic historical voting records) paired with `{#await data.streamedHistory}` in the template!

### 4.2 Progressive Enhancement with Form Actions (`use:enhance`)
- Define mutations in `+page.server.ts` `export const actions = { ... } satisfies Actions;`.
- Always validate `request.formData()` with Zod/Valibot and return `fail(422, { errors, values })` on validation failure.
- Enhance `<form method="POST" action="?/submitOrder" use:enhance>` on the client to show pending states and avoid full-page reloads while preserving zero-JS fallback resilience.

---

## 5. Svelte 5 & SvelteKit 2 Senior/Staff Checklist & Anti-Patterns

### Verification Checklist
- [ ] 100% Svelte 5 Runes mode: zero legacy `export let`, `$:`, `<slot>`, `on:click`, or `createEventDispatcher`.
- [ ] Every `{#each items as item (item.id)}` block specifies a unique key expression `(item.id)`.
- [ ] Large immutable datasets use `$state.raw()` instead of deep `$state()`.
- [ ] Shared SSR state uses `setContext`/`getContext` instantiated per request—zero module-scoped mutable `$state` singletons.
- [ ] SvelteKit 2 `fetch` provided inside `load({ fetch })` is used instead of global `window.fetch` so cookies and relative URLs work during SSR.
- [ ] `svelte-check --tsconfig ./tsconfig.json` passes with zero errors and warnings.

### Common Anti-Patterns to Reject Immediately in Code Review
1. **Mutating `$state` Inside `$derived`**: Side effects inside `$derived` are forbidden and throw runtime errors in Svelte 5.
2. **Destructuring Reactive Objects Early**: `const { count } = someStateObject;` copies the primitive value once at the time of destructuring. Access `someStateObject.count` or use `$derived`.
3. **Unkeyed `{#each}` Blocks on Dynamic Lists**: Omitting `(item.id)` causes Svelte to patch DOM nodes by index, corrupting transitions, form input focus, and child state.
4. **Exporting Global `$state` Singletons in SvelteKit SSR**: Causes cross-user data leakage between concurrent server requests.
5. **Using `$effect` for Event Handling or State Derivation**: Use event callbacks (`onclick`) for user actions and `$derived` for computed values.
