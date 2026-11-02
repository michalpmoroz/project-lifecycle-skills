# JavaScript & TypeScript Static Analysis & Lint Rules Catalog

> **Baseline Specification & Tooling Versions (Rule G)**
> - **Language & Runtime**: ECMAScript `2025` (`ES16`), TypeScript `5.7+`, Node.js `22 LTS` / Bun `1.2+` / Modern Evergreens
> - **Static Analyzers**: ESLint `9.x` (Flat Config) + `typescript-eslint` `8.x` (`strictTypeChecked` + `stylisticTypeChecked`), Biome `1.9+`, Oxlint `0.15+`

---

## 1. Mandatory `tsconfig.json` Compiler Strictness Baseline

Static lint rules are only as sound as the TypeScript type checker underneath them. Every TypeScript project must enable these non-negotiable compiler flags:

| Compiler Option | Required Value | Defect Prevented |
| :--- | :--- | :--- |
| `strict` | `true` | Enables `strictNullChecks`, `noImplicitAny`, `strictFunctionTypes`, `strictBindCallApply`, and `useUnknownInCatchVariables`. |
| `noUncheckedIndexedAccess` | `true` | Forces array lookups (`items[0]`) and dictionary reads (`map[key]`) to return `T \| undefined`, eliminating silent `TypeError: Cannot read properties of undefined` crashes. |
| `exactOptionalPropertyTypes` | `true` | Distinguishes between an omitted key (`{}`) and an explicit `{ key: undefined }`, preventing accidental `undefined` overwrites in database `PATCH` payloads. |
| `noImplicitOverride` | `true` | Requires the `override` keyword when a subclass method overrides a base class method, catching silent base-method renames at compile time. |
| `noFallthroughCasesInSwitch` | `true` | Prevents accidental fallthrough between `case` clauses in `switch` statements. |
| `noImplicitReturns` | `true` | Ensures every code path in a value-returning function explicitly returns a value or throws. |
| `verbatimModuleSyntax` | `true` | Enforces explicit `import type` and `export type` so bundlers and transpile-only tools can strip type imports deterministically without type-checking. |

---

## 2. Type Safety & Soundness Rules (13 Rules)

| # | Canonical Rule ID (Tool-Agnostic) | ESLint / Biome Equivalent | Severity | Rule Mandate & Rationale |
| :--- | :--- | :--- | :--- | :--- |
| **TS-01** | **Ban Explicit `any`** | `@typescript-eslint/no-explicit-any` / `noExplicitAny` | `error` | Never use `any`. Use `unknown` for untrusted inputs (and narrow via Zod/type guards), generics `<T>`, or specific union types. |
| **TS-02** | **Ban Unsafe `any` Propagation** | `no-unsafe-assignment`, `no-unsafe-call`, `no-unsafe-member-access`, `no-unsafe-return`, `no-unsafe-argument` | `error` | Prevents third-party or legacy `any` values from silently infecting strongly typed variables, function arguments, or return types. |
| **TS-03** | **Ban `@ts-ignore`; Require Justified `@ts-expect-error`** | `@typescript-eslint/ban-ts-comment` / `noTsIgnore` | `error` | `@ts-ignore` stays permanently even after the underlying error is fixed. Only `@ts-expect-error: <ticket + explanation>` is permitted, which fails compilation as soon as the error disappears. |
| **TS-04** | **Ban Non-Null Assertions (`!`)** | `@typescript-eslint/no-non-null-assertion` / `noNonNullAssertion` | `error` | Postfix `user!.address!.postalCode` lies to the compiler and crashes at runtime. Use optional chaining (`?.`), nullish coalescing (`??`), or an explicit guard/throw helper. |
| **TS-05** | **Enforce `unknown` on `catch` Variables** | `@typescript-eslint/use-unknown-in-catch-callback-variable` | `error` | In JS/TS, anything can be thrown (strings, numbers, `null`, DOM exceptions). Always type catch variables as `unknown` and narrow (`err instanceof Error`) before reading `.message`. |
| **TS-06** | **Exhaustive `switch` over Unions** | `@typescript-eslint/switch-exhaustiveness-check` | `error` | Every `switch` over a union or enum must handle all cases or end with a `default` branch asserting `const _exhaustive: never = value`. |
| **TS-07** | **Explicit Return Types on Exported Functions** | `@typescript-eslint/explicit-module-boundary-types` | `error` | All exported functions, service methods, and public API hooks must declare an explicit return type to prevent accidental contract drift and speed up `.d.ts` generation. |
| **TS-08** | **Enforce `import type` for Type Specifiers** | `@typescript-eslint/consistent-type-imports` / `useImportType` | `error` | Guarantees zero runtime side-effects or circular module loading for type-only dependencies. |
| **TS-09** | **Ban Double Type Assertions (`as unknown as T`)** | Custom AST rule / Code Review Gate | `error` | Casting `x as unknown as TargetType` completely bypasses structural type checking. Parse through a runtime validation schema instead. |
| **TS-10** | **Ban Unnecessary Type Assertions** | `@typescript-eslint/no-unnecessary-type-assertion` | `error` | Forbids `as T` when the expression is already of type `T`, preventing stale casts from hiding future type changes. |
| **TS-11** | **Ban Numeric TypeScript `enum`** | `no-restricted-syntax` / `noConstEnum` | `error` | Numeric TS enums allow assigning arbitrary numbers at runtime and generate bidirectional IIFE objects. Prefer union string literal types (`type OrderStatus = 'draft' \| 'paid'`) or `as const` objects. |
| **TS-12** | **Enforce Only Throwing `Error` Instances** | `@typescript-eslint/only-throw-error` (`no-throw-literal`) | `error` | Never `throw 'Failed'` or `throw { code: 400 }`. Always throw an instance of `Error` (or a custom `AppError extends Error`) to preserve stack traces and `cause` chains. |
| **TS-13** | **Ban Unbound Method References** | `@typescript-eslint/unbound-method` | `error` | Passing `service.processPayment` directly as a callback loses its `this` binding (`undefined` at call time). Wrap in an arrow function `(req) => service.processPayment(req)`. |

```typescript
// ❌ ANTI-PATTERN: Non-null assertion + unsafe `as` cast + non-exhaustive switch
export async function getClaimPayoutLabel(claimId: string) {
  const raw = (await fetchClaim(claimId)) as any;
  switch (raw!.status) {
    case 'approved':
      return `Payout: ${raw.amount}`;
    case 'denied':
      return 'Denied';
  }
}

// ✅ STAFF PATTERN: Explicit return type + runtime narrowing + exhaustive `satisfies never`
export async function getClaimPayoutLabel(claimId: ClaimId): Promise<string> {
  const claim = ClaimSchema.parse(await fetchClaim(claimId));

  switch (claim.status) {
    case 'approved':
      return `Payout: ${formatCurrencyMinorUnits(claim.payoutMinorUnits)}`;
    case 'under_review':
      return 'Pending clinical review';
    case 'denied':
      return `Denied (${claim.denialReasonCode})`;
    default: {
      const _exhaustive: never = claim.status;
      return _exhaustive;
    }
  }
}
```

---

## 3. Async, Promise & Concurrency Safety Rules (9 Rules)

Unhandled or misused Promises are the #1 source of Node.js process crashes, swallowed database errors, and race conditions.

| # | Canonical Rule ID (Tool-Agnostic) | ESLint / Biome Equivalent | Severity | Rule Mandate & Rationale |
| :--- | :--- | :--- | :--- | :--- |
| **AS-01** | **Ban Floating Promises** | `@typescript-eslint/no-floating-promises` / `noFloatingPromises` | `error` | Every call returning a `Promise` must be `await`ed, `return`ed, `.catch()`ed, or explicitly marked with `void` (only when fire-and-forget is intentional and internally error-handled). |
| **AS-02** | **Ban Misused Promises** | `@typescript-eslint/no-misused-promises` / `noMisusedPromises` | `error` | Forbids passing an `async` function where a `void` return is expected (e.g., `array.forEach(async () => ...)` or `if (asyncCheck())`, where a `Promise` object is always truthy!). |
| **AS-03** | **Mandatory `return await` in `try/catch`** | `@typescript-eslint/return-await` (`in-try-catch`) | `error` | Inside a `try { ... } catch` block, you **must** write `return await asyncOp()`. Returning an un-awaited Promise exits the `try` block before the Promise rejects, bypassing the local `catch` block completely! |
| **AS-04** | **Ban `async` Promise Executors** | `no-async-promise-executor` / `noAsyncPromiseExecutor` | `error` | Never write `new Promise(async (resolve, reject) => ...)`. Rejected promises or thrown errors inside an `async` executor are not caught by the `Promise` constructor. |
| **AS-05** | **Ban `await` on Non-Thenables** | `@typescript-eslint/await-thenable` | `error` | Flags `await syncFunction()`, which often reveals a misunderstanding that a synchronous function is asynchronous. |
| **AS-06** | **Ban `async` Functions Without `await`** | `@typescript-eslint/require-await` / `useAwait` | `error` | Do not mark functions `async` unless they contain an `await` expression or explicitly implement an async interface contract. |
| **AS-07** | **Reject Promises Only With `Error` Objects** | `@typescript-eslint/prefer-promise-reject-errors` | `error` | Ensures `Promise.reject(new AppError(...))` always provides a stack trace. |
| **AS-08** | **Ban Sequential `await` in Loops for Independent I/O** | `no-await-in-loop` (with batch concurrency pattern) | `warn` / `error` | Sequential `for (const id of ids) { await fetchItem(id); }` turns 20 fast 50ms queries into a 1,000ms waterfall. Use `await Promise.all(...)` or a bounded concurrency pool (`p-limit`). |
| **AS-09** | **Ban Atomic State Updates Across `await` Gaps** | `require-atomic-updates` | `error` | Prevents `total += await computeDelta()` where `total` was read before `await` and overwrites concurrent updates that occurred while paused. |

```typescript
// ❌ ANTI-PATTERN: Missing `return await` inside `try/catch` + `if (asyncFn())` truthy bug
async function verifyAndChargeOrder(orderId: OrderId): Promise<ChargeReceipt> {
  // BUG 1: `isFraudFlagged(orderId)` returns a Promise<boolean>, which is ALWAYS truthy!
  if (isFraudFlagged(orderId)) {
    throw new FraudBlockedError(orderId);
  }

  try {
    // BUG 2: Without `return await`, a rejection from `gateway.charge` bypasses this catch block!
    return gateway.charge(orderId);
  } catch (err: unknown) {
    logger.error({ orderId, err }, 'Payment gateway charge failed');
    throw new PaymentGatewayError('Unable to complete charge', { cause: err });
  }
}

// ✅ STAFF PATTERN: Awaited boolean condition + explicit `return await` in `try/catch`
async function verifyAndChargeOrder(orderId: OrderId): Promise<ChargeReceipt> {
  if (await isFraudFlagged(orderId)) {
    throw new FraudBlockedError(orderId);
  }

  try {
    return await gateway.charge(orderId);
  } catch (err: unknown) {
    logger.error({ orderId, err }, 'Payment gateway charge failed');
    throw new PaymentGatewayError('Unable to complete charge', { cause: err });
  }
}
```

---

## 4. Immutability, Variables & Data Integrity Rules (8 Rules)

| # | Canonical Rule ID (Tool-Agnostic) | ESLint / Biome Equivalent | Severity | Rule Mandate & Rationale |
| :--- | :--- | :--- | :--- | :--- |
| **IM-01** | **Enforce `const` by Default; Ban `var`** | `prefer-const`, `no-var` / `useConst`, `noVar` | `error` | Declare all bindings with `const`. Use `let` only for loop counters or accumulators where reassignment is genuinely cleaner than `.reduce()` / `.map()`. Never use `var`. |
| **IM-02** | **Ban Function Parameter Mutation** | `no-param-reassign` (`{ props: true }`) / `noParameterAssign` | `error` | Never mutate an incoming function parameter or its properties (`order.total = 0`). Return a new object (`{ ...order, totalMinorUnits: 0 }`) so callers never suffer spooky action-at-a-distance side effects. |
| **IM-03** | **Prefer Immutable ES2023+ Array Methods** | Custom / `eslint-plugin-unicorn` | `error` | `.sort()`, `.reverse()`, and `.splice()` mutate the original array in place — a notorious source of React/Angular/Vue state bugs. Always use `.toSorted()`, `.toReversed()`, `.toSpliced()`, and `.with(index, value)`. |
| **IM-04** | **Enforce `readonly` Properties & `ReadonlyArray<T>`** | `@typescript-eslint/prefer-readonly` | `error` | Mark class properties, domain DTO fields, and array inputs (`readonly items: readonly LineItem[]`) as `readonly` unless mutation is explicitly required. |
| **IM-05** | **Ban Variable Shadowing** | `@typescript-eslint/no-shadow` / `noShadowRestrictedNames` | `error` | Never declare a local variable (`user`, `err`, `res`) with the same name as a variable in an enclosing scope, which causes developers to read/write the wrong binding. |
| **IM-06** | **Ban Unused Variables & Parameters** | `@typescript-eslint/no-unused-vars` / `noUnusedVariables` | `error` | Dead variables clutter code. Intentionally unused positional parameters (e.g., `_req` in `(_req, res) => ...`) must be prefixed with an underscore `_`. |
| **IM-07** | **Ban `delete` on Object Properties & Arrays** | `@typescript-eslint/no-dynamic-delete`, `@typescript-eslint/no-array-delete` | `error` | `delete obj.prop` mutates V8 hidden classes (deoptimizing objects into slow dictionary mode) and `delete arr[i]` creates sparse arrays with `empty` holes. Use destructuring omission (`const { secret, ...publicProfile } = user`), `Map.delete()`, or `arr.filter()`. |
| **IM-08** | **Prefer Object Spread & Destructuring** | `prefer-object-spread`, `prefer-destructuring` | `error` | Prefer `{ ...defaults, ...overrides }` over `Object.assign({}, defaults, overrides)`. |

---

## 5. Operators, Equality, Coercion & Security Guardrails (12 Rules)

| # | Canonical Rule ID (Tool-Agnostic) | ESLint / Biome Equivalent | Severity | Rule Mandate & Rationale |
| :--- | :--- | :--- | :--- | :--- |
| **OP-01** | **Enforce Strict Equality (`===` / `!==`)** | `eqeqeq` (`always`) / `noDoubleEquals` | `error` | Never use loose equality (`==` or `!=`), even for `null` checks. Explicitly write `value === null \|\| value === undefined` (or narrow via `??` / `?.`). |
| **OP-02** | **Prefer Nullish Coalescing (`??`) Over Logical OR (`\|\|`)** | `@typescript-eslint/prefer-nullish-coalescing` | `error` | Using `const copay = inputCopay \|\| 20` overwrites a valid `0` copay or `""` string with the fallback! Always use `inputCopay ?? 20`, which only falls back on `null` or `undefined`. |
| **OP-03** | **Prefer Optional Chaining (`?.`)** | `@typescript-eslint/prefer-optional-chain` / `useOptionalChain` | `error` | Replace verbose `patient && patient.insurance && patient.insurance.policyNumber` with `patient?.insurance?.policyNumber`. |
| **OP-04** | **Strict Boolean Expressions on Nullable Numbers & Strings** | `@typescript-eslint/strict-boolean-expressions` | `error` | Never write `if (count)` or `{items.length && <List />}` (which renders a literal `"0"` into the DOM in React JSX!). Write explicit predicates: `if (count > 0)`, `if (name.length > 0)`, `if (user !== null)`. |
| **OP-05** | **Ban Implicit Type Coercion Tricks** | `no-implicit-coercion` | `error` | Ban `+str`, `"" + num`, `!!val` (in ambiguous contexts), and `~indexOf()`. Use explicit `Number(str)`, `String(num)`, `Boolean(val)`, and `.includes()`. |
| **OP-06** | **Ban `eval()`, `new Function()`, and String Timer Callbacks** | `no-eval`, `no-implied-eval`, `no-new-func` | `error` | Remote code execution (RCE) and XSS vectors; violates strict Content Security Policy (`CSP`). |
| **OP-07** | **Ban `for..in` Loops Over Arrays or Unfiltered Objects** | `@typescript-eslint/no-for-in-array`, `guard-for-in` | `error` | `for..in` iterates over inherited prototype keys and stringified array indices. Use `for (const item of items)` for arrays and `for (const [key, val] of Object.entries(obj))` for objects. |
| **OP-08** | **Ban Unexplained Magic Numbers** | `@typescript-eslint/no-magic-numbers` (ignore `-1, 0, 1, 2`) | `warn` / `error` | Extract domain thresholds (`86_400_000`, `429`, `0.0825`) into named constants (`MS_PER_DAY`, `HTTP_TOO_MANY_REQUESTS`, `STATE_SALES_TAX_RATIO`). Use numeric separators (`1_000_000`) for readability. |
| **OP-09** | **Ban `console.log` in Production Source** | `no-console` | `error` | Leftover `console.log` leaks PII and clutters stdout. Use a structured logger (`pino`, `winston`) on the backend or an explicit telemetry wrapper on the frontend. |
| **OP-10** | **Enforce `Number.isNaN()` and `Number.isFinite()`** | `unicorn/prefer-number-properties` / `useNumberNamespace` | `error` | Global `isNaN('abc')` coerces non-numbers and returns `true`. Always use `Number.isNaN(val)` and `Number.isFinite(val)`. |
| **OP-11** | **Enforce Radix or Explicit Parsing on `parseInt`** | `radix` / `useParseIntRadix` | `error` | Always pass radix `10` (`Number.parseInt(str, 10)`) or use `Number(str)`. |
| **OP-12** | **Enforce `switch` Default Clause (`default-case-last`)** | `default-case`, `default-case-last` | `error` | Every `switch` statement must place its `default:` branch last and either handle unexpected values or assert exhaustiveness. |

```tsx
// ❌ ANTI-PATTERN: Logical OR `||` clobbering `0` + JSX `{count && <Component />}` rendering `"0"` + mutating `.sort()`
export function VoterTurnoutSummary({ reportedPrecincts, overrideTurnoutPercent, precincts }: Props) {
  const displayPercent = overrideTurnoutPercent || 50; // BUG: If override is `0`, displays `50`!
  const sorted = precincts.sort((a, b) => b.votes - a.votes); // BUG: Mutates prop array in place!

  return (
    <section>
      <p>Turnout: {displayPercent}%</p>
      {reportedPrecincts && <PrecinctTable rows={sorted} />} {/* BUG: Renders literal "0" when 0! */}
    </section>
  );
}

// ✅ STAFF PATTERN: Nullish coalescing `??` + explicit `> 0` boolean check + immutable `.toSorted()`
export function VoterTurnoutSummary({
  reportedPrecincts,
  overrideTurnoutPercent,
  precincts,
}: Props): React.JSX.Element {
  const displayPercent = overrideTurnoutPercent ?? 50;
  const sortedPrecincts = precincts.toSorted((a, b) => b.votes - a.votes);

  return (
    <section>
      <p>Turnout: {displayPercent}%</p>
      {reportedPrecincts > 0 ? <PrecinctTable rows={sortedPrecincts} /> : <EmptyPrecinctState />}
    </section>
  );
}
```
