# React 19 & Next.js 15 Performance Guide

> **Baseline Framework & Tooling Versions**:
> - **React Core**: React 19.x (`babel-plugin-react-compiler`, Server Components, Actions, `useTransition`, `useDeferredValue`, `useOptimistic`, `React.cache`)
> - **Meta-Framework**: Next.js 15.x (App Router, Streaming SSR, Partial Prerendering, `next/dynamic`, `next/image`, `next/font`)
> - **Virtualization & State**: `@tanstack/react-virtual` 3.x, Zustand 5.x / Jotai 2.x
> - **Core Web Vitals Targets**: LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1

---

## 1. React Compiler (Auto-Memoization) vs. Manual Memoization

- **Rule**: In React 19 projects with `babel-plugin-react-compiler` enabled, write clean, idiomatic React following the Rules of React (pure render functions, no mutating props/state in render) and let the compiler automatically memoize values, callbacks, and JSX elements.
- **When Manual `useMemo` / `useCallback` / `React.memo` Is Still Required**:
  - In codebases without React Compiler enabled.
  - When passing callbacks or objects into custom hooks or third-party imperative libraries where referential stability controls `useEffect` execution (e.g., WebSocket reconnection effects or map SDK re-initialization).
- **Anti-Pattern (Context Value Thrashing)**: Passing an inline object literal `<AuthContext.Provider value={{ user, login, logout }}>` without memoization (when not using React Compiler) forces every consumer in the tree to re-render whenever the provider's parent re-renders. Split high-frequency state (e.g., cursor coordinates, live price ticks) from low-frequency state (theme, authenticated user) into separate Contexts or use an external atomic store (Zustand / Jotai).

---

## 2. React Server Components (RSC) Payload & Serialization Budgets

- **Rule**: Keep Server Components as the default in Next.js App Router. Only add `'use client'` at the **leaf interactive nodes** (buttons, search inputs, client carousels) rather than at page or layout roots.
- **Serialization Pitfall**: Never pass an entire 50-field database entity or raw ORM record from a Server Component down as props to a `'use client'` component.
- **Why**: Every prop crossing the Server-to-Client boundary is serialized twice in the initial HTML stream (once in rendered HTML, once in the `__next_f` RSC payload script tags), doubling HTML transfer size and hydration parse time. Map records down to strict DTOs containing only the 3–5 fields the client component actually reads.

```tsx
// ANTI-PATTERN: Serializing massive raw DB records across the RSC -> Client boundary
// <ClientPatientRow patient={fullPatientRecordWith500LabNotes} />

// BEST PRACTICE: Project only required scalar fields in the Server Component
export default async function PatientRosterPage() {
  const patients = await fetchPatientsSummary();
  return (
    <section>
      {patients.map((p) => (
        <ClientPatientActions
          key={p.id}
          patientId={p.id}
          displayName={p.displayName}
          triageStatus={p.triageStatus}
        />
      ))}
    </section>
  );
}
```

---

## 3. Per-Request Server Deduplication with `React.cache()`

- **Rule**: Wrap database queries, ORM calls, and non-`fetch` SDK functions in `React.cache()` when they are called across multiple Server Components or inside both `generateMetadata()` and the page tree during the same request.
- **Why**: While Next.js automatically deduplicates identical `GET` `fetch()` calls within a single render pass, direct database/ORM queries (Prisma, Drizzle, `pg`) and gRPC/SDK calls are **not** deduplicated unless wrapped in `React.cache()`.

```tsx
// lib/queries/get-listing.ts
import { cache } from 'react';
import { db } from '@/lib/db';

// Deduplicated per HTTP request across generateMetadata(), Layout, and Page Server Components!
export const getPropertyListing = cache(async (listingId: string) => {
  return db.query.listings.findFirst({
    where: (l, { eq }) => eq(l.id, listingId),
    columns: { id: true, title: true, priceCents: true, city: true, heroImageUrl: true },
  });
});
```

---

## 4. Eliminating Client-Side Fetch Waterfalls & Parallelizing Server Loads

- **Rule**: Never chain sequential `await` calls for independent data sources inside Server Components or Route Handlers, and avoid `useEffect` fetch waterfalls on the client. Use `Promise.all()` or `<Suspense>` streaming boundaries so fast UI shells paint immediately while slower data streams in.

```tsx
// GOOD: Parallel data initiation + Streaming Suspense boundary
import { Suspense } from 'react';

export default async function ProductDetailPage({ params }: { params: Promise<{ sku: string }> }) {
  const { sku } = await params;
  // Critical data awaited for initial shell
  const product = await getProductCore(sku);

  return (
    <main>
      <ProductHero product={product} />
      {/* Non-blocking streaming boundary for slow recommendations/reviews */}
      <Suspense fallback={<ReviewsSkeleton />}>
        <ProductReviewsSection sku={sku} />
      </Suspense>
    </main>
  );
}
```

---

## 5. Non-Blocking UI Updates (`useTransition` & `useDeferredValue`)

- **Rule**: Wrap expensive non-urgent state updates (e.g., filtering a large E-commerce catalog, re-rendering a complex FinTech chart on range change, switching dashboard tabs) inside `startTransition()` or derive a lagged value with `useDeferredValue()`.
- **Impact**: Keeps the text input or button click responsive (low INP) because React can interrupt the background re-render if the user types another keystroke.

```tsx
'use client';
import { useState, useDeferredValue } from 'react';

export function SearchableDirectory({ items }: { items: DirectoryItem[] }) {
  const [query, setQuery] = useState('');
  // Urgent: input updates immediately. Deferred: heavy list filtering lags gracefully without blocking typing.
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;

  return (
    <div>
      <input
        type="search"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Filter records..."
      />
      <div style={{ opacity: isStale ? 0.7 : 1, transition: 'opacity 150ms' }}>
        <HeavyResultsTable query={deferredQuery} items={items} />
      </div>
    </div>
  );
}
```

---

## 6. Instant Perceived Latency with `useOptimistic()` (React 19)

- **Rule**: For user-initiated mutations with predictable outcomes (e.g., bookmarking a listing, toggling a clinical task status, adding an item to cart, or voting on a civic issue), use React 19's `useOptimistic()` hook to reflect the new state on screen within **< 16ms** (1 frame) before the Server Action or API mutation resolves.
- **Why**: Eliminates perceived network round-trip latency while automatically rolling back to the authoritative server state if the mutation fails.

```tsx
'use client';
import { useOptimistic, useTransition } from 'react';

export function FavoriteToggleButton({
  itemId,
  isFavoritedServer,
  toggleFavoriteAction,
}: {
  itemId: string;
  isFavoritedServer: boolean;
  toggleFavoriteAction: (id: string, nextState: boolean) => Promise<void>;
}) {
  const [isPending, startTransition] = useTransition();
  const [optimisticFavorite, setOptimisticFavorite] = useOptimistic(
    isFavoritedServer,
    (_currentState, optimisticValue: boolean) => optimisticValue
  );

  function handleClick() {
    const nextValue = !optimisticFavorite;
    startTransition(async () => {
      setOptimisticFavorite(nextValue); // Paints immediately in next frame!
      await toggleFavoriteAction(itemId, nextValue);
    });
  }

  return (
    <button
      type="button"
      onClick={handleClick}
      aria-pressed={optimisticFavorite}
      disabled={isPending && false /* keep interactive */}
    >
      {optimisticFavorite ? 'Saved' : 'Save'}
    </button>
  );
}
```

---

## 7. Lazy-Loading Client-Only Heavy Widgets (`next/dynamic` with `ssr: false`)

- **Rule**: Wrap heavy browser-only interactive components—such as WebGL/Three.js canvases, geospatial maps (MapLibre/Leaflet), rich-text editors (TipTap/Monaco), and PDF viewers—in `next/dynamic` with `{ ssr: false }` and a fixed-aspect-ratio `loading` skeleton.
- **Why**: Prevents the server from wasting CPU attempting to render canvas/window-dependent components during SSR and splits their 200 KB–800 KB JavaScript payload out of the critical initial hydration bundle.

```tsx
'use client';
import dynamic from 'next/dynamic';

const InteractiveElectoralMap = dynamic(
  () => import('./InteractiveElectoralMap').then((mod) => mod.InteractiveElectoralMap),
  {
    ssr: false,
    loading: () => (
      <div
        className="map-placeholder-skeleton"
        style={{ aspectRatio: '16 / 9', width: '100%' }}
        aria-busy="true"
      />
    ),
  }
);
```

---

## 8. List Virtualization (`@tanstack/react-virtual`) & Stable `key` Rules

- **Rule**: Never use array index (`key={index}`) as a React `key` on lists where items can be inserted, deleted, filtered, or reordered. Always use a stable domain ID (`key={order.id}`).
- **Rule**: Virtualize any list > 100 items using `@tanstack/react-virtual` (lightweight ~3 KB headless hook).

---

## 9. React 19 & Next.js 15 Quick Audit Matrix

| Bottleneck / Anti-Pattern | React 19 / Next.js 15 Remediation |
| :--- | :--- |
| **Cascading Context Re-Renders** | Enable `babel-plugin-react-compiler`, split high/low-frequency Contexts, or use Zustand/Jotai selectors. |
| **Bloated HTML / RSC `__next_f` Payload** | Keep `'use client'` at leaf nodes; project only 3–5 required scalar fields across the RSC boundary. |
| **Duplicate DB Queries in Metadata + Page** | Wrap server DB/ORM queries in `React.cache()`. |
| **Sequential `await` Waterfalls** | Initiate promises in parallel (`Promise.all`) and stream slow sections in `<Suspense>`. |
| **Input Lag During Filtering (Poor INP)** | Wrap non-urgent state updates in `startTransition()` or `useDeferredValue()`. |
| **Sluggish Mutation Feedback** | Apply optimistic UI updates via `useOptimistic()`. |
| **Heavy Map/Chart Bundle Blocking Load** | Load via `next/dynamic(() => import(...), { ssr: false, loading: ... })`. |
| **LCP Image Regression** | Use `<Image src="..." width={...} height={...} priority sizes="..." />`. |
