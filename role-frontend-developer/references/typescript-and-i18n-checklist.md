# Strict TypeScript & Internationalization (i18n/l10n/RTL/Timezones) Checklist (`references/typescript-and-i18n-checklist.md`)

> **Baseline Versions**: TypeScript `5.7+` (`strict: true`, `noUncheckedIndexedAccess: true`), Zod `3.24+` / Valibot `1.0+`, Unicode CLDR / ICU MessageFormat, ECMAScript `Intl` APIs, IANA Time Zone Database (tzdb).
> **Mandate**: Load and apply this checklist on all TypeScript modeling, boundary validation, localization, RTL layout, and date/time/currency engineering tasks.

> [!NOTE]
> **Continuous Evolution Mandate**: This checklist is a baseline quality floor, never a ceiling. Proactively enforce compile-time invariants and locale/timezone correctness across every user-facing and API-facing boundary.

---

## 1. Strict TypeScript Architecture (TypeScript 5.7+)

### 1.1 Non-Negotiable `tsconfig.json` Compiler Flags
Every frontend workspace should enforce:
```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,
    "verbatimModuleSyntax": true
  }
}
```
- With `noUncheckedIndexedAccess: true`, indexing an array `items[0]` or record `dict[key]` returns `T | undefined`, eliminating `TypeError: Cannot read properties of undefined` in production.

### 1.2 Discriminated Unions & Exhaustiveness Checking
Never model mutually exclusive UI states with loose boolean bags (`isLoading`, `isError`, `data`). Use discriminated unions:

```ts
type AsyncDataState<T> =
  | { readonly status: 'idle' }
  | { readonly status: 'loading' }
  | { readonly status: 'success'; readonly data: T; readonly updatedAt: Date }
  | { readonly status: 'error'; readonly code: string; readonly retryable: boolean };

function renderStatusBadge<T>(state: AsyncDataState<T>): string {
  switch (state.status) {
    case 'idle':
      return 'Ready';
    case 'loading':
      return 'Syncing...';
    case 'success':
      return `Updated ${state.updatedAt.toISOString()}`;
    case 'error':
      return `Error (${state.code})`;
    default:
      // Compile-time error if a new status variant is added and not handled here!
      return state satisfies never;
  }
}
```

### 1.3 Branded (Nominal) Types for Domain Safety
Prevent catastrophic parameter-order bugs (such as passing a `PatientId` where a `MedicationId` is expected, or a `CandidateId` where a `DistrictId` is expected) using zero-runtime-cost branded types:

```ts
declare const __brand: unique symbol;
export type Brand<T, B extends string> = T & { readonly [__brand]: B };

export type PatientId = Brand<string, 'PatientId'>;
export type EncounterId = Brand<string, 'EncounterId'>;
export type CentsAmount = Brand<number, 'CentsAmount'>;
```

### 1.4 `const` Type Parameters & `NoInfer<T>` Utility Type
- **`const` Type Parameters**: Use `const` on generic type parameters (`<const T extends ...>`) in configuration builders, route tables, and permission definitions so TypeScript automatically infers narrow literal types without requiring callers to append `as const`:
  ```ts
  interface NavRoute<TPath extends string = string> {
    readonly path: TPath;
    readonly roles: readonly ('admin' | 'clinician' | 'auditor' | 'customer')[];
  }

  export function defineRoutes<const TRoutes extends readonly NavRoute[]>(routes: TRoutes): TRoutes {
    return routes;
  }

  // Inferred type of `routes[0].path` is literal `"/patients/:id"`, not widened `string`!
  const routes = defineRoutes([{ path: '/patients/:id', roles: ['clinician'] }]);
  ```
- **`NoInfer<T>` Intrinsic Type**: Prevent TypeScript from widening a constrained union generic parameter based on a secondary argument (such as a fallback or initial value):
  ```ts
  export function createStepWizard<TStep extends string>(
    steps: readonly TStep[],
    initialStep: NoInfer<TStep>,
  ) {
    // Passing an invalid `initialStep` not present in `steps` is now a compile-time error
    // instead of widening `TStep` to `string`!
    return { steps, current: initialStep };
  }
  ```

### 1.5 Runtime Boundary Validation (Zod / Valibot)
TypeScript types evaporate at compile time. Always validate untrusted payloads at system boundaries:
- `fetch` / API responses (when not guaranteed by an end-to-end codegen contract)
- `JSON.parse(localStorage.getItem(...))`
- URL query parameters (`URLSearchParams`)
- `window.postMessage` & WebSocket frames

---

## 2. Internationalization (i18n), Localization (l10n), RTL & Timezones

### 2.1 ICU MessageFormat for Plurals & Gender/Select
Never hardcode English plural logic (`count === 1 ? 'item' : 'items'`). Slavic, Semitic, and Celtic languages have 3 to 6 plural forms (`zero`, `one`, `two`, `few`, `many`, `other`).
```txt
# ICU MessageFormat Example (Multi-Domain: E-commerce / Healthcare / Civic)
{count, plural,
  =0 {No matching records found in {region}.}
  one {# record found in {region}.}
  few {# records found in {region}.}
  many {# records found in {region}.}
  other {# records found in {region}.}
}
```

### 2.2 Native `Intl` Formatting (Zero Moment.js Bloat)
Always use standard browser `Intl` formatters with the user's active locale:
- **Currency**: `new Intl.NumberFormat(locale, { style: 'currency', currency: 'EUR' }).format(amount)`
- **Dates & Timezones**: `new Intl.DateTimeFormat(locale, { dateStyle: 'medium', timeStyle: 'short', timeZone: userTz }).format(date)`
- **Relative Time**: `new Intl.RelativeTimeFormat(locale, { numeric: 'auto' }).format(-2, 'day')`
- **Locale-Aware Sorting**: `items.toSorted((a, b) => new Intl.Collator(locale, { sensitivity: 'base', numeric: true }).compare(a.name, b.name))`

### 2.3 Timezone-Safe ISO-8601 & IANA Timezone Engineering
Confusing UTC instants with local calendar dates causes off-by-one-day bugs across timezones and Daylight Saving Time (DST) transitions. Always classify every temporal value into one of three distinct representations:

| Temporal Category | Multi-Domain Examples | Wire & Storage Format | Frontend Handling Rule |
| :--- | :--- | :--- | :--- |
| **1. Exact Past/Present Instant** | FinTech transaction executed, E-commerce order placed, Audit log timestamp. | **UTC ISO-8601 Instant**: `"2026-11-02T14:30:00.000Z"` | Store and transmit in UTC (`Z`). Format for display using `Intl.DateTimeFormat` with the viewer's target `timeZone`. |
| **2. Civil / Wall-Clock Date (Timezone-Less)** | Patient date of birth, Civic election day, Holiday, Insurance coverage start date. | **ISO-8601 Calendar Date**: `"2026-11-03"` (`YYYY-MM-DD`, **no time or `Z`!**) | **Never** pass `"2026-11-03"` into `new Date("2026-11-03")` and format in local time (which shifts to `"2026-11-02"` in `America/New_York` UTC-5!). Parse as year/month/day components or format with `timeZone: 'UTC'`. |
| **3. Future Scheduled Local Event** | Healthcare clinic appointment (`09:00` in clinic's city), Airline departure, Recurring webinar. | **Local ISO Date-Time + Explicit IANA Zone**: `{ localDateTime: "2026-11-15T09:00:00", timeZone: "America/New_York" }` (or `"Europe/Warsaw"`) | Store the IANA timezone identifier (`America/New_York`, never fixed numeric offsets like `-05:00`) alongside the wall-clock time so legislative or DST transitions never shift the local appointment hour. |

### 2.4 Bidirectional (BiDi) & RTL Layout Rules
- Set `<html lang="ar" dir="rtl">` dynamically when switching to an RTL locale (Arabic, Hebrew, Urdu, Persian).
- **Use `dir="auto"` on user-generated content inputs and comments** so mixed LTR/RTL paragraphs align naturally.
- **Icon Mirroring Rules**:
  - **Mirror (`transform: scaleX(-1)` in `[dir="rtl"]`)**: Directional arrows (back/next chevrons, breadcrumbs, external link arrows, progress bars, sidebar layouts).
  - **Do NOT Mirror**: Media playback controls (Play/Pause/Fast-Forward always point right), clock faces, checkmarks, brand logos, phone receivers, and magnifying glasses.
- **Text Expansion Resilience**: German, Finnish, Polish, and Greek translations are often **30%–40% longer** than English. Never use fixed pixel widths (`width: 90px`) or `white-space: nowrap` without overflow handling on buttons, tabs, or navigation badges.
