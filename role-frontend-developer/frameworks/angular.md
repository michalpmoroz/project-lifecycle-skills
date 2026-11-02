# Angular Senior/Staff Engineering Guide (`frameworks/angular.md`)

> **Baseline Versions**: Angular `19.x` / `18.x`, TypeScript `5.7+`, RxJS `7.8+`, NgRx SignalStore `19.x`, Angular CLI / Esbuild (`application` builder).
> **Mandate**: This guide provides the mandatory baseline for modern Angular applications. Apply these patterns by default and continuously evaluate newer stable Angular primitives as the platform evolves.

---

## 1. Standalone-by-Default & Modular Architecture

- **Zero NgModules for New Code**: In Angular 19+, `standalone: true` is the default for components, directives, and pipes (in Angular 18, set `standalone: true` explicitly; in Angular 19, omit the redundant `standalone: true` property unless project lint rules require it). Never introduce new `@NgModule` classes.
- **Explicit Granular Imports**: Import only the exact standalone components, directives, and pipes used by a component's template. Never import massive barrel modules or legacy `SharedModule` bundles that bloat lazy chunks.
- **Host Element Binding via `host` Metadata**: Always use the `host: { ... }` property inside `@Component` / `@Directive` decorators instead of `@HostBinding` and `@HostListener` decorators.

```typescript
// GOOD: Modern Angular 19 Standalone Component with host bindings & OnPush
@Component({
  selector: 'app-metric-card',
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [DecimalPipe, CurrencyPipe],
  host: {
    'class': 'metric-card',
    '[class.metric-card--critical]': 'isCritical()',
    '[attr.aria-busy]': 'isLoading()',
    '(keydown.enter)': 'activate()',
  },
  template: `...`
})
export class MetricCardComponent { ... }
```

---

## 2. Fine-Grained Reactivity with Signals

Signals are the primary reactive primitive for synchronous state, derived state, and template rendering in Angular.

### 2.1 `signal()`, `computed()`, and `linkedSignal()`
- **`signal<T>(initialValue)`**: Use for mutable local component or service state. Expose read-only references publicly (`readonly state = this.#state.asReadonly()`) to prevent external mutation outside domain methods.
- **`computed(() => ...)`**: Use for all derived values. `computed` is lazy, memoized, and side-effect-free. **Never** mutate state or trigger HTTP requests inside `computed()`.
- **`linkedSignal()` (Angular 19+)**: Use when local mutable state needs to automatically reset whenever an upstream source signal changes (e.g., resetting a selected shipping option when an E-commerce cart's country changes, or resetting active pagination page when a Civic candidate search query updates).

```typescript
// Multi-domain example: E-commerce Variant Picker / Healthcare Slot Selector / Civic District Filter
export class CatalogFilterComponent {
  readonly availableRegions = input.required<Region[]>();
  
  // Automatically resets to the first region whenever `availableRegions` input changes,
  // while still allowing the user to manually override the selection via `.set()`.
  readonly selectedRegion = linkedSignal(() => this.availableRegions()[0] ?? null);
}
```

### 2.2 Declarative Async State with `resource()` and `rxResource()` (Angular 19+)
- Use `rxResource()` (when integrating with `HttpClient` / RxJS streams) or `resource()` (Promise-based with `AbortSignal`) to connect reactive Signal parameters to asynchronous data fetching.
- Always handle `.isLoading()`, `.error()`, and `.value()` states in the template, and leverage automatic request cancellation when reactive request parameters change.

```typescript
@Injectable({ providedIn: 'root' })
export class PatientRecordsFacade {
  readonly #http = inject(HttpClient);
  readonly selectedPatientId = signal<string | null>(null);

  readonly patientResource = rxResource({
    request: () => ({ id: this.selectedPatientId() }),
    loader: ({ request }) => {
      if (!request.id) return of(null);
      return this.#http.get<PatientRecord>(`/api/v1/patients/${encodeURIComponent(request.id)}`);
    },
  });
}
```

### 2.3 Strict Discipline Around `effect()`
- **`effect()` is an escape hatch, not a state synchronization tool.**
- **PROHIBITED**: Using `effect()` to copy value from one signal into another signal (use `computed()` or `linkedSignal()` instead).
- **VALID USES**: Synchronizing state with non-Angular DOM APIs (Canvas, Charts, 3rd-party map SDKs like MapLibre, `localStorage` persistence, analytics logging, focus management).

---

## 3. Signal-Based Component APIs

Legacy decorator-based APIs (`@Input()`, `@Output()`, `@ViewChild()`, `@ContentChild()`) are prohibited in new components.

| Legacy Decorator (Prohibited) | Modern Signal Primitive (Mandatory) | Notes |
| :--- | :--- | :--- |
| `@Input() title = '';` | `readonly title = input('');` | Returns `InputSignal<string>` |
| `@Input({ required: true }) id!: string;` | `readonly id = input.required<string>();` | Enforced at compile time in templates |
| `@Input() + @Output() valueChange` | `readonly value = model<string>('');` | Two-way bindable `ModelSignal<T>` |
| `@Output() saved = new EventEmitter<Item>();` | `readonly saved = output<Item>();` | Lightweight `OutputEmitterRef<T>` (no RxJS overhead) |
| `@ViewChild('chart') chart?: ElementRef;` | `readonly chart = viewChild.required<ElementRef>('chart');` | Reactive `Signal<ElementRef>` |
| `@ContentChildren(TabItem)` | `readonly tabs = contentChildren(TabItem);` | Reactive `Signal<readonly TabItem[]>` |

- Remember that `input()`, `model()`, and `viewChild()` remove the need for `ngOnChanges` and `ngAfterViewInit` boilerplate in 95% of cases—derive state directly via `computed()`.

---

## 4. `ChangeDetectionStrategy.OnPush` & Zoneless Readiness

- **Mandatory `OnPush`**: Every single component must declare `changeDetection: ChangeDetectionStrategy.OnPush`.
- **Zoneless Architecture (`provideExperimentalZonelessChangeDetection`)**:
  - Write all components assuming `zone.js` does **not** exist.
  - Never rely on monkey-patched `setTimeout`, `Promise.then`, or DOM event listeners mutating plain class properties to trigger UI refreshes.
  - Ensure all template-bound state lives in **Signals** (`signal()`, `computed()`, `toSignal()`) or triggers explicit `ChangeDetectorRef.markForCheck()` when bridging legacy third-party callbacks.

---

## 5. Built-In Control Flow & `@defer` Lazy Loading

Legacy structural directives (`*ngIf`, `*ngFor`, `*ngSwitch`) are prohibited. Use Angular's built-in block syntax:

### 5.1 `@if`, `@for`, `@switch`, and `@let`
- **`@for` Mandatory `track`**: Always provide a unique, stable identity expression (`track item.id`).
  - **Anti-Pattern**: Using `track $index` on dynamic collections that can be filtered, sorted, inserted, or deleted (causes DOM state corruption and unnecessary node recreation). Use `track $index` **only** for static, immutable primitives.
- **Use `@empty` block**: Always pair `@for` over user-facing lists with an `@empty` block to render an accessible empty state.
- **Use `@let` for template variables**: Use `@let user = currentUser();` to avoid repeated signal invocations or null assertions in templates.

```html
@let summary = accountSummary();
@if (isLoading()) {
  <app-skeleton-table aria-busy="true" />
} @else if (error(); as err) {
  <app-error-banner [message]="err.userMessage" (retry)="reload()" />
} @else {
  @for (tx of transactions(); track tx.transactionId) {
    <app-transaction-row [transaction]="tx" />
  } @empty {
    <app-empty-state
      title="No transactions found"
      description="Adjust your date range or status filters to see ledger activity." />
  }
}
```

### 5.2 Declarative Partial Loading with `@defer`
- Wrap heavy below-the-fold components (e.g., E-commerce product review histograms, FinTech interactive candlestick charts, Civic district boundary maps, SaaS comment threads) in `@defer`.
- Always provide `@placeholder` (with explicit dimensions to prevent CLS!), `@loading (minimum 150ms; after 100ms)` (to avoid loading flicker), and `@error` blocks.

```html
@defer (on viewport; prefetch on idle) {
  <app-geospatial-district-map [boundaries]="districtGeoJson()" />
} @placeholder {
  <div class="h-[420px] w-full rounded-lg bg-surface-subtle" aria-hidden="true"></div>
} @loading (after 100ms; minimum 300ms) {
  <div class="h-[420px] w-full animate-pulse rounded-lg bg-surface-muted" role="status">
    <span class="sr-only">Loading interactive map...</span>
  </div>
} @error {
  <p role="alert" class="p-4 text-error">Unable to load map module. Please check your connection.</p>
}
```

---

## 6. Pure Pipes vs. Template Method Calls

- **Never Call Unmemoized Methods in Templates**: Calling `{{ formatRiskScore(item) }}` or `[items]="filterItems(rawItems)"` inside a template executes on every change detection pass unless backed by a `computed()` signal or a **Pure Pipe** (`pure: true`, the default).
- Prefer `computed()` signals for component-specific derivations and standalone `@Pipe({ name: '...', pure: true })` for reusable formatting transformations across components.

---

## 7. Modern Dependency Injection (`inject()`)

- **Prefer `inject()` over Constructor Injection**: Use `readonly #service = inject(MyService);` with native ECMAScript private fields (`#`) instead of `constructor(private service: MyService)`. This simplifies inheritance, works seamlessly in functional guards/interceptors, and enables reusable composition functions.
- **Typed `InjectionToken<T>`**: Never use string tokens. Always use `new InjectionToken<AppConfig>('APP_CONFIG')` for configuration, feature flags, and abstract ports.
- **Environment Initializers**: Use `provideAppInitializer(() => ...)` (Angular 19+, replacing `APP_INITIALIZER`) for startup telemetry or configuration hydration.

---

## 8. Strictly Typed Reactive Forms

- **Use `NonNullableFormBuilder`**: Always inject `NonNullableFormBuilder` (or set `{ nonNullable: true }`) so `form.reset()` restores initial default values instead of `null`.
- **Explicit Form Interfaces**: Define explicit typed form interfaces (`FormGroup<{ email: FormControl<string>; amount: FormControl<number> }>`) for complex forms (e.g., FinTech wire transfer, Healthcare intake form, E-commerce checkout).
- **Bridge Forms to Signals with `toSignal()`**: When deriving UI state from form changes, wrap `form.valueChanges` or `form.statusChanges` with `toSignal(this.form.valueChanges, { initialValue: this.form.getRawValue() })`.

---

## 9. Routing, Functional Guards & View Transitions

- **Router Configuration (`app.config.ts`)**:
  - Enable `withComponentInputBinding()` so route parameters (`:id`), query parameters (`?page=2`), and route data bind directly to component `input()` signals without injecting `ActivatedRoute` in every component!
  - Enable `withViewTransitions()` for smooth native browser View Transitions between routes.
  - Enable `withInMemoryScrolling({ scrollPositionRestoration: 'enabled', anchorScrolling: 'enabled' })`.
- **Lazy Loading Everything**: Every route must use `loadComponent: () => import('./feature.component').then(m => m.FeatureComponent)` or `loadChildren` to keep the initial main bundle minimal.
- **Functional Guards & Resolvers**: Class-based guards (`implements CanActivate`) are deprecated. Write pure functional guards (`CanActivateFn`, `CanMatchFn`):

```typescript
export const requireRoleGuard = (allowedRole: UserRole): CanMatchFn => {
  return () => {
    const authStore = inject(AuthStore);
    const router = inject(Router);
    return authStore.hasRole(allowedRole)
      ? true
      : new RedirectCommand(router.parseUrl('/unauthorized'));
  };
};
```

---

## 10. RxJS Interop & Subscription Hygiene

- **Signals for State, RxJS for Events & Streams**:
  - Use **Signals** for synchronous state and template values.
  - Use **RxJS** for complex asynchronous event coordination (debounced typeahead, websocket multiplexing, retry with exponential backoff, race-condition cancellation).
- **Bridge Primitives (`@angular/core/rxjs-interop`)**:
  - `toSignal(observable$, { initialValue })`: Automatically subscribes and unsubscribes when the component/service is destroyed. Never use `AsyncPipe` in new templates when `toSignal()` can provide synchronous access everywhere in the component.
  - `toObservable(signal)`: Converts a signal into an Observable stream for RxJS operator pipelines (e.g., `debounceTime`, `switchMap`).
  - `takeUntilDestroyed(this.#destroyRef)`: Mandatory on every manual `.subscribe()` call.
- **Flattening Operator Discipline**:
  - `switchMap`: Cancels in-flight request when a new emission arrives (use for search/filter reads).
  - `exhaustMap`: Ignores new emissions while current request is in flight (use for form submissions / login / payment buttons to prevent double-submit).
  - `concatMap`: Queues emissions sequentially in order (use for ordered sequential writes).
  - `mergeMap`: Runs concurrently in parallel (avoid for mutations unless order does not matter).

---

## 11. Functional HTTP Interceptors & `HttpClient`

Configure `provideHttpClient(withFetch(), withInterceptors([authInterceptor, errorRetryInterceptor]))`.

```typescript
export const correlationAndAuthInterceptor: HttpInterceptorFn = (req, next) => {
  const tokenService = inject(TokenStorageService);
  const token = tokenService.accessToken();

  const cloned = req.clone({
    setHeaders: {
      'X-Request-ID': crypto.randomUUID(),
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
    },
  });
  return next(cloned);
};
```

---

## 12. Server-Side Rendering (SSR) & Incremental Hydration

- **Incremental Hydration & Event Replay (Angular 19+)**:
  - Configure `provideClientHydration(withIncrementalHydration(), withEventReplay())`.
  - Annotate `@defer (hydrate on interaction)` or `@defer (hydrate on viewport)` so heavy islands remain static HTML until the user interacts with or scrolls to them, while `withEventReplay()` captures early clicks and replays them post-hydration.
- **Platform Safety (`afterNextRender` / `afterRender`)**:
  - **NEVER** access `window`, `document`, `localStorage`, or `navigator` in constructors, `ngOnInit`, or root service initializers—they crash Node/Edge SSR.
  - Wrap browser-only DOM manipulation inside `afterNextRender(() => { ... })`.
- **DOM Structure Integrity**: Ensure valid HTML nesting (e.g., never place a `<div>` inside a `<p>` or omit `<tbody>` in `<table>`), as browser auto-correction will cause SSR hydration mismatches (`NG0500`).

---

## 13. NgRx SignalStore for Complex Domain State

For shared feature or global domain state (e.g., E-commerce cart & checkout orchestrator, Healthcare clinical charting workspace, SaaS multi-tenant dashboard), prefer `@ngrx/signals` (`signalStore`, `withState`, `withComputed`, `withMethods`, `withHooks`) over heavyweight Redux boilerplate when event-sourcing is not required.

---

## 14. Angular Senior/Staff Verification Checklist & Anti-Patterns

### Verification Checklist
- [ ] All components use `ChangeDetectionStrategy.OnPush` and standalone architecture.
- [ ] Inputs, outputs, and view queries use `input()`, `output()`, `model()`, and `viewChild()`.
- [ ] Templates use `@if`, `@for (...; track item.id)`, `@switch`, and `@defer`—zero legacy `*ngIf` / `*ngFor`.
- [ ] Zero uncleaned `.subscribe()` calls; all streams use `toSignal()`, `rxResource()`, or `takeUntilDestroyed()`.
- [ ] `withComponentInputBinding()` and functional guards/interceptors are used across routing and HTTP layers.
- [ ] SSR safety verified: all browser-specific APIs are gated inside `afterNextRender()`.

### Common Anti-Patterns to Reject Immediately in Code Review
1. **Mutating Signals In-Place**: Calling `this.items().push(newItem)` (mutates reference without notifying consumers). Always use `this.items.update(list => [...list, newItem])`.
2. **Writing to Signals Inside `effect()`**: Creates infinite loops, glitchy intermediate frames, and untraceable data flow. Use `computed()` or `linkedSignal()`.
3. **Using `track $index` on Mutable Lists**: Breaks animations, focus state, and child component state when rows are added/removed/sorted.
4. **Calling `.subscribe()` Inside Another `.subscribe()`**: Creates callback hell, race conditions, and memory leaks. Compose streams with `switchMap`, `concatMap`, or `exhaustMap`.
5. **Importing `CommonModule` Everywhere**: Imports unnecessary directives; built-in `@if`/`@for` need zero imports, and pipes (`DatePipe`, `CurrencyPipe`) should be imported individually.
