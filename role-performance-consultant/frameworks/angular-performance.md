# Angular Performance Guide (Angular 18 / 19+)

> **Baseline Framework & Tooling Versions**:
> - **Framework**: Angular 19.x / 18.x (Standalone Components, Signals, `linkedSignal`, `resource` / `rxResource` / `httpResource`, Zoneless Change Detection, `@defer`, Incremental Hydration)
> - **Reactive Streams & Compiler**: RxJS 7.8+, Angular CLI / Vite + esbuild Application Builder, TypeScript 5.7+
> - **Core Web Vitals Targets**: LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1

---

## 1. `ChangeDetectionStrategy.OnPush` & Zoneless Angular

- **Rule**: Every Angular component must explicitly set `changeDetection: ChangeDetectionStrategy.OnPush` (or run inside a Zoneless application using `provideExperimentalZonelessChangeDetection()` / `provideZonelessChangeDetection()`).
- **Why**: Default `Zone.js` (`ChangeDetectionStrategy.Default`) monkey-patches every browser event, timer, and Promise to dirty-check the **entire component tree** top-to-bottom on every microtask. Zoneless + Signals eliminates `zone.js` (~13 KB gzipped payload savings) and schedules targeted view updates strictly for components reading dirtied signals.

```typescript
// app.config.ts (Angular 19+)
import { ApplicationConfig, provideExperimentalZonelessChangeDetection } from '@angular/core';
import { provideClientHydration, withIncrementalHydration } from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [
    provideExperimentalZonelessChangeDetection(),
    provideClientHydration(withIncrementalHydration()),
  ],
};
```

---

## 2. Fine-Grained Reactivity with Signals (`signal`, `computed`, `linkedSignal`)

- **Rule**: Use `signal()` for synchronous reactive state and `computed()` for memoized derived state.
- **Anti-Pattern (Calling Methods in Templates)**: Never bind a method call like `{{ calculateTotalTax(cartItems) }}` or `{{ formatDiagnosis(record) }}` directly in an Angular template. Template expressions execute on **every** change detection cycle. Use `computed()` signals or **Pure `@Pipe({ pure: true })`**.
- **Anti-Pattern (Overusing `effect()`)**: Never use `effect()` to synchronize one piece of state into another (`effect(() => this.filtered.set(...))`), which causes cascading double renders. Use `computed()` for read-only derivation or `linkedSignal()` (Angular 19+) for writable state that resets when a source signal changes.

```typescript
import { Component, ChangeDetectionStrategy, input, computed, linkedSignal } from '@angular/core';

@Component({
  selector: 'app-catalog-summary',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="summary-card">
      <span>Total ({{ itemCount() }} items): {{ formattedTotal() }}</span>
    </div>
  `,
})
export class CatalogSummaryComponent {
  // Signal inputs (Angular 18/19+)
  readonly items = input.required<ReadonlyArray<{ id: string; priceCents: number; qty: number }>>();
  readonly currency = input<string>('USD');

  // Memoized: only recalculates when items() reference or currency() changes
  readonly itemCount = computed(() =>
    this.items().reduce((sum, item) => sum + item.qty, 0)
  );

  readonly formattedTotal = computed(() => {
    const cents = this.items().reduce((sum, item) => sum + item.priceCents * item.qty, 0);
    return new Intl.NumberFormat('en-US', { style: 'currency', currency: this.currency() }).format(cents / 100);
  });

  // Writable state that automatically resets to the first item when items() changes
  readonly selectedItemId = linkedSignal(() => this.items()[0]?.id ?? null);
}
```

---

## 3. Declarative Async Data Fetching & Cancellation (`resource`, `rxResource`, `httpResource`)

- **Rule**: Use Angular 19+ `resource()`, `rxResource()`, or `httpResource()` to bind async data fetching directly to reactive Signal parameters, and **always wire up the provided `abortSignal`**.
- **Why**: When a user types rapidly into a search box or switches filters (e.g., filtering E-commerce products, querying Healthcare ICD-10 codes, or switching FinTech portfolio timeframes), `resource()` automatically aborts the previous in-flight `fetch`/`HttpClient` request before starting the new one—preventing out-of-order race conditions and wasted network/server load.

```typescript
import { Component, ChangeDetectionStrategy, signal, resource } from '@angular/core';

@Component({
  selector: 'app-patient-lookup',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <input
      type="search"
      [value]="searchQuery()"
      (input)="searchQuery.set($any($event.target).value)"
      placeholder="Search patient registry..."
    />
    @if (patientsResource.isLoading()) {
      <p aria-busy="true">Searching...</p>
    }
    <ul>
      @for (patient of patientsResource.value() ?? []; track patient.id) {
        <li>{{ patient.fullName }} — {{ patient.mrn }}</li>
      }
    </ul>
  `,
})
export class PatientLookupComponent {
  readonly searchQuery = signal('');

  readonly patientsResource = resource({
    request: () => ({ query: this.searchQuery().trim() }),
    loader: async ({ request, abortSignal }) => {
      if (!request.query) return [];
      // Automatically cancels previous in-flight HTTP request when searchQuery() changes!
      const res = await fetch(`/api/v1/patients?q=${encodeURIComponent(request.query)}`, {
        signal: abortSignal,
      });
      return res.json() as Promise<Array<{ id: string; fullName: string; mrn: string }>>;
    },
  });
}
```

---

## 4. Escaping Zone Pollution (`NgZone.runOutsideAngular`)

- **Rule**: In applications that still use `Zone.js` (not yet fully Zoneless), always attach high-frequency event listeners (`scroll`, `mousemove`, `pointermove`, `touchmove`, `requestAnimationFrame` loops, WebGL/Chart.js renderers, or high-frequency WebSocket market ticks) inside `this.ngZone.runOutsideAngular(() => ...)`.
- **Why**: Without `runOutsideAngular`, every single `pointermove` or animation frame (60–120 times per second) triggers `Zone.js` to run `ApplicationRef.tick()` across the component tree, causing severe main-thread CPU saturation and degraded INP. Re-enter the zone (`this.ngZone.run(() => ...)`) only when a meaningful threshold is crossed that requires a template update.

```typescript
import { Component, ElementRef, NgZone, OnInit, inject, DestroyRef } from '@angular/core';

@Component({
  selector: 'app-interactive-chart-canvas',
  template: `<canvas #chartCanvas></canvas>`,
})
export class InteractiveChartCanvasComponent implements OnInit {
  private readonly ngZone = inject(NgZone);
  private readonly hostRef = inject(ElementRef<HTMLElement>);
  private readonly destroyRef = inject(DestroyRef);

  ngOnInit(): void {
    const controller = new AbortController();
    this.destroyRef.onDestroy(() => controller.abort());

    // High-frequency pointer tracking runs outside Angular change detection
    this.ngZone.runOutsideAngular(() => {
      this.hostRef.nativeElement.addEventListener(
        'pointermove',
        (event: PointerEvent) => {
          this.renderCrosshairOnCanvas(event.offsetX, event.offsetY);
        },
        { passive: true, signal: controller.signal }
      );
    });
  }

  private renderCrosshairOnCanvas(_x: number, _y: number): void {
    // Direct canvas 2D/WebGL draw — zero Angular change detection overhead
  }
}
```

---

## 5. Phased DOM Reads & Writes (`afterNextRender` / `afterRender`)

- **Rule**: When measuring DOM geometry (`getBoundingClientRect()`, `offsetWidth`) and mutating styles to integrate third-party libraries or position popovers, use Angular's `afterNextRender` or `afterRender` with explicit phase separation (`earlyRead` vs. `write`) instead of `ngAfterViewInit` + `setTimeout`.
- **Why**: Angular batches all `earlyRead` callbacks across all components first, and then executes all `write` callbacks together, completely eliminating Forced Synchronous Layout (layout thrashing).

```typescript
import { Component, ElementRef, afterNextRender, inject } from '@angular/core';

@Component({
  selector: 'app-auto-sized-panel',
  template: `<div class="panel-content"><ng-content /></div>`,
})
export class AutoSizedPanelComponent {
  private readonly el = inject(ElementRef<HTMLElement>);

  constructor() {
    afterNextRender({
      earlyRead: () => {
        // 1. Read DOM geometry before any component mutates styles
        return this.el.nativeElement.getBoundingClientRect().width;
      },
      write: (measuredWidth) => {
        // 2. Mutate DOM safely using the value returned from earlyRead
        this.el.nativeElement.style.setProperty('--panel-width', `${measuredWidth}px`);
      },
    });
  }
}
```

---

## 6. Control Flow `@for` Mandatory `track` Expression

- **Rule**: In `@for (item of items(); track item.id)`, always track by a **unique, immutable domain identifier** (`item.id`, `item.sku`, `item.patientId`, `item.precinctCode`).
- **Anti-Pattern**: Using `track $index` on lists that can be filtered, sorted, or prepended, or `track item` (object identity) when polling replaces object references from an API. Tracking by object identity destroys and recreates all DOM nodes on every API refresh.

---

## 7. Deferrable Views (`@defer`) & Incremental Hydration (`withIncrementalHydration()`)

- **Rule**: Wrap heavy below-the-fold or conditionally visible components (charts, map canvases, comment threads, rich modals) in `@defer` blocks with `on viewport` or `on interaction` triggers and `prefetch on idle`.
- **Rule (Angular 19 Incremental Hydration)**: Combine `@defer (hydrate on viewport)` or `@defer (hydrate on interaction)` with `withIncrementalHydration()` so SSR renders the HTML immediately for SEO/LCP, while the component's JavaScript bundle is neither downloaded nor hydrated until the user actually scrolls to or interacts with it.

```html
<!-- SSR renders the placeholder/server HTML immediately; JS loads & hydrates only on interaction -->
@defer (on viewport; prefetch on idle; hydrate on interaction) {
  <app-interactive-district-map [geoData]="districtBoundaries()" />
} @placeholder (minimum 300ms) {
  <div class="map-skeleton" style="aspect-ratio: 16 / 9; width: 100%;"></div>
} @loading (after 100ms; minimum 400ms) {
  <div class="map-spinner" aria-busy="true">Loading interactive map...</div>
}
```

---

## 8. `NgOptimizedImage` Directive (`ngSrc`)

- **Rule**: Always use `NgOptimizedImage` (`<img ngSrc="..." width="..." height="..." />`) instead of raw `src`.
- **Benefits**: Enforces explicit dimensions to prevent CLS, automatically sets `fetchpriority="high"` and injects a `<link rel="preload">` tag when `priority` is added to the LCP image, generates responsive `srcset` URLs, and warns in development if an LCP image is missing `priority`.

---

## 9. RxJS Subscription Hygiene (`takeUntilDestroyed` & `shareReplay`)

- **Rule 1**: Always prefer the `AsyncPipe` or `toSignal(observable$)` over manual `.subscribe()`. When manual `.subscribe()` is unavoidable, pipe `takeUntilDestroyed(this.destroyRef)` to prevent memory leaks.
- **Rule 2 (`shareReplay` Trap)**: When caching shared observables with `shareReplay`, **always** specify `{ bufferSize: 1, refCount: true }` unless the stream is truly a permanent application-lifetime singleton. `shareReplay(1)` defaults to `refCount: false`, keeping the upstream subscription and cached payload alive in memory forever even after all subscribers unsubscribe.

---

## 10. Angular Performance Quick Audit Matrix

| Bottleneck / Anti-Pattern | Angular 18 / 19+ Remediation |
| :--- | :--- |
| **Full-Tree Change Detection** | Enable `ChangeDetectionStrategy.OnPush` everywhere + migrate to `provideExperimentalZonelessChangeDetection()`. |
| **Template Method Calls (`{{ getTotal() }}`)** | Convert to memoized `computed()` Signal or pure `@Pipe({ pure: true })`. |
| **High-Frequency Event Lag (`pointermove`/`scroll`)** | Run listener inside `ngZone.runOutsideAngular()` or use Zoneless change detection. |
| **DOM Thrashing in Lifecycle Hooks** | Separate measurements and mutations using `afterNextRender({ earlyRead, write })`. |
| **Stale / Duplicate Search Requests** | Use `resource()` / `rxResource()` / `httpResource()` and pass `abortSignal` to cancel superseded requests. |
| **DOM Re-Creation on List Refresh** | Use `@for (item of items(); track item.id)` with a stable primary key (never `track item`). |
| **Hydration Main-Thread Blocking** | Enable `withIncrementalHydration()` and wrap below-the-fold widgets in `@defer (hydrate on viewport)`. |
| **LCP Image Delay & CLS** | Use `<img ngSrc="/hero.avif" width="1200" height="800" priority />`. |
