# React & Next.js Senior/Staff Engineering Guide (`frameworks/react.md`)

> **Baseline Versions**: React `19.x`, Next.js `15.x` (App Router), TypeScript `5.7+`, TanStack Query `5.x`, Zustand `5.x`, `nuqs` `2.x`.
> **Mandate**: This guide defines the mandatory baseline for modern React and Next.js engineering. Apply these idioms by default and continuously adapt to newer stable React and compiler advancements.

---

## 1. React 19 Core Primitives & Idioms

### 1.1 `ref` as a Standard Prop & `<Context>` Provider
- **No More `forwardRef`**: In React 19, function components accept `ref` directly as a standard prop and support cleanup functions returned from ref callbacks.
- **Direct `<Context value={...}>`**: Render `<ThemeContext value={theme}>` directly instead of the legacy `<ThemeContext.Provider value={theme}>`.

```tsx
import type { ComponentPropsWithRef } from 'react';

interface ButtonProps extends ComponentPropsWithRef<'button'> {
  variant?: 'primary' | 'secondary' | 'danger';
}

export function ActionButton({ variant = 'primary', ref, children, ...props }: ButtonProps) {
  return (
    <button ref={ref} data-variant={variant} {...props}>
      {children}
    </button>
  );
}
```

### 1.2 Actions, `useActionState`, `useFormStatus`, and `useOptimistic`
- **`useActionState`**: Manage form/mutation submission lifecycle (pending state, structured field errors, returned result) natively with progressive enhancement support.
- **`useFormStatus`**: Read parent `<form>` pending state inside reusable submit buttons without prop-drilling `isSubmitting`.
- **`useOptimistic`**: Provide instant, flicker-free optimistic UI feedback (e.g., E-commerce wishlist toggle, Healthcare task completion checkbox, Civic poll bookmark, SaaS Kanban card drag) that automatically rolls back if the server action fails.

```tsx
'use client';
import { useActionState, useOptimistic } from 'react';
import { submitTransferAction, type TransferState } from './actions';

const initialState: TransferState = { status: 'idle' };

export function WireTransferForm({ currentBalance }: { currentBalance: number }) {
  const [state, formAction, isPending] = useActionState(submitTransferAction, initialState);
  const [optimisticBalance, addOptimisticDeduction] = useOptimistic(
    currentBalance,
    (balance, amount: number) => balance - amount
  );

  return (
    <form
      action={async (formData) => {
        const amount = Number(formData.get('amount') ?? 0);
        addOptimisticDeduction(amount);
        await formAction(formData);
      }}
      aria-describedby={state.status === 'error' ? 'transfer-error' : undefined}
    >
      <p className="tabular-nums">Available Balance: ${optimisticBalance.toFixed(2)}</p>
      <input name="amount" type="number" inputMode="decimal" required min={1} />
      {state.status === 'error' && (
        <p id="transfer-error" role="alert" className="text-error">
          {state.message}
        </p>
      )}
      <button type="submit" disabled={isPending}>
        {isPending ? 'Processing Transfer...' : 'Send Funds'}
      </button>
    </form>
  );
}
```

### 1.3 The `use()` API
- Use `use(promise)` to unwrap promises streamed from Server Components into Client Components paired with `<Suspense>` and `<ErrorBoundary>`.
- Unlike hooks (`useContext`), `use(Context)` may be called conditionally inside `if` blocks and loops after early returns.

---

## 2. React Server Components (RSC) vs. Client Components

### 2.1 Boundary Discipline (`'use client'`)
- **Server-First Default**: In Next.js App Router, keep components as Server Components by default so data fetching executes close to the database/API and zero JavaScript for static trees is shipped to the client.
- **Push `'use client'` to the Leaves**: Never mark an entire page or layout `'use client'` just to add an interactive search bar or theme toggle. Extract only the interactive island into a dedicated Client Component and pass server-rendered content via `children` (the **Slot / Donut Pattern**).
- **Serialization Rules Across the RSC Boundary**: Props passed from a Server Component to a Client Component must be serializable (plain objects, primitives, arrays, Promises, or Server Actions). Never pass class instances, unserialized `Date` methods, or plain functions (other than `'use server'` actions) across the boundary.

### 2.2 Server Actions (`'use server'`) Security Mandate
- **Every `'use server'` function is a public HTTP POST endpoint.**
- **Mandatory 3-Step Guard inside Every Server Action**:
  1. **Authenticate & Authorize**: Verify the current user session and role permissions inside the action itself (never assume the caller came from a protected page).
  2. **Validate Input Schema**: Parse `FormData` or arguments with Zod/Valibot before touching any database or downstream service.
  3. **Return Structured Result Objects**: Return discriminated unions (`{ status: 'success', data } | { status: 'error', fieldErrors }`) for expected validation failures instead of throwing unhandled exceptions that trigger generic error pages.

---

## 3. Next.js 15 App Router Specifics

### 3.1 Asynchronous Dynamic APIs (Next.js 15 Breaking Change)
In Next.js 15, `params`, `searchParams`, `cookies()`, `headers()`, and `draftMode()` are **asynchronous Promises** and must be awaited (or unwrapped via `use(params)` in Client Components).

```tsx
// Next.js 15 Page Signature (E-commerce Product / Civic Candidate / Healthcare Provider Page)
interface DetailPageProps {
  params: Promise<{ slug: string }>;
  searchParams: Promise<{ tab?: string; page?: string }>;
}

export default async function DetailPage({ params, searchParams }: DetailPageProps) {
  const { slug } = await params;
  const { tab = 'overview', page = '1' } = await searchParams;
  // ...
}
```

### 3.2 Caching Semantics in Next.js 15
- **Uncached by Default**: `fetch()` requests, `GET` Route Handlers, and Client Router Cache entries are **no longer cached by default** in Next.js 15.
- Opt into caching deliberately using `fetch(url, { cache: 'force-cache', next: { tags: ['catalog'] } })` or Next.js 15's `'use cache'` directive, and invalidate precisely via `revalidateTag('catalog')` or `revalidatePath(...)`.
- **Avoiding Server Waterfalls**: When a Server Component needs multiple independent datasets, initiate fetches in parallel using `Promise.all([getProfile(id), getActivity(id)])` or split them into sibling Server Components wrapped in independent `<Suspense>` boundaries for progressive streaming.

---

## 4. Eliminating `useEffect` Abuse

> [!CAUTION]
> **90% of `useEffect` usages in typical codebases are anti-patterns** that cause extra render passes, state tearing, race conditions, and hydration bugs.

| Anti-Pattern (`useEffect` Abuse) | Correct Senior/Staff Pattern |
| :--- | :--- |
| Updating state B inside `useEffect` when state/prop A changes | **Compute during render**: `const fullName = `${firstName} ${lastName}`;` (or `useMemo` if computationally expensive: `> 1ms`). |
| Resetting all state inside `useEffect` when an `id` prop changes | **Pass a `key` prop** to the component: `<PatientEditor key={patient.id} patient={patient} />`. React cleanly remounts with fresh state. |
| Triggering an API POST or analytics call inside `useEffect` after a button sets a boolean flag | **Execute side effects inside the event handler** (`onClick` / `onSubmit`) that caused the interaction. |
| Fetching data inside `useEffect` without cancellation or cache deduplication | Use **Server Components**, **TanStack Query (`useQuery`)**, or **React 19 `use(promise)`**. |
| Subscribing to browser APIs (`matchMedia`, `online`, `localStorage`) via `useEffect` + `useState` | Use **`useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot)`** to prevent tearing and SSR hydration mismatches. |

---

## 5. State Management Architecture

### 5.1 URL State First (`nuqs`)
- Any state that controls filtering, sorting, search queries, pagination, active tabs, or selected modal items should live in the URL query string using `nuqs` (`useQueryState`, `useQueryStates`) with type-safe parsers (`parseAsInteger`, `parseAsStringEnum`).

### 5.2 Server Cache State (TanStack Query v5)
When client-side fetching, polling, or infinite scrolling is required:
- **Query Key Factories**: Centralize hierarchical query keys (`invoiceKeys.detail(id)`, `invoiceKeys.list(filters)`) so cache invalidation is deterministic.
- **No Destructured Property Loss**: Use `useSuspenseQuery` paired with `<Suspense>` or handle `status === 'pending' | 'error' | 'success'` exhaustively.
- **Throw on Error / Cancellation**: Always forward the `signal` provided by `queryFn: ({ signal }) => fetch(..., { signal })` so abandoned queries abort cleanly.

### 5.3 Shared Client State (Zustand v5)
- Keep Zustand stores small and domain-scoped.
- **Atomic Selectors Mandatory**: Always select specific slices (`const itemCount = useCartStore((s) => s.items.length)`) or use `useShallow`. **Never** call `const store = useCartStore()` without a selector, which subscribes the component to re-render on *every* state change in the entire store.

---

## 6. Custom Hooks Hygiene

- **When to Extract a Custom Hook**: Only extract a `use*` hook when encapsulating stateful lifecycle logic or external subscriptions. If a function does not call any React hooks, make it a plain, pure TypeScript utility function!
- **Return Stable Contracts**: Return objects (`{ data, isLoading, reset }`) for hooks with `> 2` return values, or tuples (`[value, setValue] as const`) for `useState`-like primitives.

---

## 7. Resilience: Suspense & Error Boundaries

- Every route segment and major asynchronous feature widget (e.g., FinTech live ticker, E-commerce recommendation carousel, Healthcare lab results panel) must be isolated inside its own `<ErrorBoundary>` (or `error.tsx`) and `<Suspense fallback={<Skeleton />}>` (or `loading.tsx`).
- A failure in a secondary widget must **never** crash the entire page shell.

---

## 8. Strict TypeScript Patterns for React

- **Extend Native HTML Attributes**: Use `ComponentPropsWithoutRef<'input'>` or `ComponentPropsWithRef<'button'>` so custom wrappers forward `aria-*`, `data-*`, `id`, `disabled`, and event handlers natively.
- **Discriminated Union Props**: Use discriminated unions when prop combinations are mutually exclusive:

```tsx
// A link-button must either have `href` (renders <a>) OR `onClick` (renders <button>), never neither or half-configured.
type ActionTriggerProps =
  | ({ as: 'link'; href: string } & ComponentPropsWithRef<'a'>)
  | ({ as?: 'button'; href?: never } & ComponentPropsWithRef<'button'>);
```

---

## 9. React & Next.js Senior/Staff Checklist & Anti-Patterns

### Verification Checklist
- [ ] `'use client'` boundaries are pushed down to interactive leaf components; Server Components handle initial data loads.
- [ ] All Next.js 15 `params`, `searchParams`, `cookies()`, and `headers()` are properly awaited.
- [ ] Every Server Action (`'use server'`) verifies authentication/authorization and validates inputs via Zod/Valibot.
- [ ] Zero `useEffect` hooks used for derived state, event handling chains, or uncancelled `fetch` calls.
- [ ] List renders use stable unique domain IDs (`key={item.id}`), never array indices on dynamic lists.
- [ ] Forms support keyboard submission, native validation/ARIA error binding, and disabled/pending states via `useActionState` / `useFormStatus`.

### Common Anti-Patterns to Reject Immediately in Code Review
1. **Defining Components Inside Other Components**: `function Parent() { function Child() { ... } }` causes React to unmount and remount `Child` from scratch on every single render of `Parent`, destroying DOM focus and state.
2. **Derived State Stored in `useState` + Synced via `useEffect`**: Causes double renders and stale intermediate states.
3. **Leaking Server Secrets to Client Components**: Passing full database user records (including password hashes or internal flags) as props across the `'use client'` boundary exposes them in the serialized RSC payload in the browser HTML/network tab. Map to a strict DTO first!
4. **Whole-Store Zustand Subscriptions**: `const { a, b } = useStore()` without `useShallow` or atomic selectors.
5. **Sequential `await` Waterfalls in Server Components**: Awaiting 4 independent queries sequentially instead of using `Promise.all` or independent `<Suspense>` boundaries.
