# Universal Formatting, Naming Conventions & File Structure Rules

> **Baseline Specification & Tooling Versions (Rule G)**
> - **Character Encoding & Line Endings**: Unicode `UTF-8` (without BOM), POSIX `LF` (`\n`)
> - **Language Specifications**: ECMAScript `2025` (`ES16`), TypeScript `5.7+`, Python `3.13+` (`PEP 8`), Go `1.23+` (`gofmt`), Rust `2024 Edition` (`rustfmt`)
> - **Formatters & Structural Linters**: EditorConfig `0.16+`, Prettier `3.4+` / Biome `1.9+`, ESLint `9.x` (`eslint-plugin-import-x` / `perfectionist`), Ruff `0.8+`

---

## 1. Text, Whitespace & Syntax Formatting Rules

Every source file across the repository must obey these deterministic text-formatting invariants (translated into `.editorconfig`, Prettier, or Biome during Phase 6 project scaffolding):

| Rule Category | Baseline Standard | Rationale & Scope |
| :--- | :--- | :--- |
| **Character Encoding** | `UTF-8` (no BOM) | Prevents byte-order-mark corruption in shell scripts, JSON parsers, and HTTP headers. |
| **Line Endings** | `LF` (`\n`) everywhere | Enforced via `.editorconfig` and `.gitattributes` (`* text=auto eol=lf`) so Windows (`CRLF`) and Unix/macOS developers never produce full-file line-ending diffs. |
| **Indentation Style & Width** | **2 spaces**: TS, JS, JSX/TSX, HTML, CSS/SCSS, JSON, YAML, Markdown, SQL<br>**4 spaces**: Python, Rust, Java, C#, Kotlin<br>**Hard Tabs**: Go (`gofmt`), `Makefile` | Matches ecosystem standards while keeping component trees and declarative markup readable without horizontal scrolling. |
| **Print Width (Line Length)** | **100 characters** target soft wrap; **120 characters** hard ceiling | Enables side-by-side split-screen diff reviews on standard laptops without horizontal scrollbars. (Exception: long URLs, raw regex literals, or SQL migration strings). |
| **Trailing Whitespace** | Strictly stripped (`trim_trailing_whitespace = true`) | Eliminates invisible diff noise across commits. (Exception: `*.md` only if two-space hard line breaks are explicitly used, though `<br>` or blank lines are preferred). |
| **Final Newline (EOF)** | Mandatory single `LF` at end of every file (`insert_final_newline = true`) | POSIX standard requirement; prevents `\`No newline at end of file\`` warnings in `git diff` and concatenation bugs in POSIX CLI tools. |
| **Quote Style** | **Single quotes (`'...'`)**: JS, TS, CSS, SCSS<br>**Double quotes (`"..."`)**: HTML attributes, JSX/TSX attributes, JSON | Minimizes visual clutter in JS/TS while escaping double quotes inside HTML strings cleanly. Template literals (`` `...` ``) are used only when interpolating expressions (`${...}`) or preserving multiline strings. |
| **Semicolons (JS/TS)** | **Always explicit (`;`)** | Prevents Automatic Semicolon Insertion (ASI) hazards when lines begin with `(`, `[`, `` ` ``, `+`, `-`, or `/`. |
| **Trailing Commas** | **Always on multiline (`all`)** | Mandatory on multiline object literals, array literals, function parameter lists, and import blocks so adding or reordering the final item touches only 1 line in `git diff` instead of 2. |
| **Brace Style & Control Blocks** | **1TBS (`One True Brace Style` / K&R)** with **mandatory curly braces** | Opening brace on the same line (`if (cond) {`). **Never** omit curly braces for single-line `if`, `else`, `for`, or `while` statements (prevents `goto fail`-style indentation bugs). |
| **Arrow Function Parentheses** | **Always (`(x) => x * 2`)** | Uniform parameter syntax whether there is 1 parameter, type annotations `(x: number) => ...`, or destructuring. |
| **Operator Line Breaks** | **Before the operator** (`?`, `:`, `&&`, `||`, `+`) at the start of the continuation line | Placing the operator at the beginning of the wrapped line makes it immediately obvious that the line is a continuation of an expression. |
| **Ternary Expressions** | Single-level only; **strictly ban nested ternaries** | Nested ternaries (`a ? b : c ? d : e`) destroy readability. Replace with an explicit `if / else` helper function, `switch` block, or dictionary lookup map. |

```typescript
// ❌ ANTI-PATTERN: Braceless single-line `if` + nested ternary + missing trailing comma
if (!user.isActive) return null;
const tierLabel = user.points > 1000 ? 'Platinum' : user.points > 500 ? 'Gold' : 'Standard';

// ✅ STAFF PATTERN: Mandatory braces + flat lookup/helper + clean multiline trailing commas
if (!user.isActive) {
  return null;
}

function resolveLoyaltyTier(points: number): LoyaltyTier {
  if (points > 1000) {
    return 'Platinum';
  }
  if (points > 500) {
    return 'Gold';
  }
  return 'Standard';
}
```

---

## 2. Cross-Language Naming Conventions Matrix

Every identifier must conform to both the casing convention of its syntactic construct and the Ubiquitous Language documented in `<workspace>/docs/domain/glossary.md`.

### 2.1 Casing & Role Matrix

| Construct | Casing Standard | Multi-Domain Examples (E-commerce, FinTech, Healthcare, Civic, SaaS) | Anti-Patterns to Ban |
| :--- | :--- | :--- | :--- |
| **Directories & File Names** | `kebab-case` + explicit architectural role suffix | `payment-intent.service.ts`, `patient-vitals-card.component.tsx`, `ballot-measure.schema.ts`, `invoice-reconciliation.spec.ts` | `PaymentIntentService.ts`, `patient_vitals.tsx`, `utils2.ts`, `index.ts` (containing business logic) |
| **Variables & Properties** | `camelCase` (JS/TS/Go/Java)<br>`snake_case` (Python/Rust/SQL) | `cartSubtotalMinorUnits`, `clinicalEncounterId`, `precinctTurnoutRate`, `tenantSeatLimit` | `data`, `info`, `item`, `val`, `temp`, `obj`, `res2`, `d` |
| **Booleans (Variables, Props, Functions)** | `camelCase` / `snake_case` with **mandatory positive predicate prefix**: `is*`, `has*`, `can*`, `should*`, `will*`, `did*` | `isEmailVerified`, `hasActiveSubscription`, `canApproveTransfer`, `shouldRetryWebhook`, `didConsentToCookies` | Negative names (`isNotHidden`, `disableTls = false`, `noRetry`) or noun-only booleans (`active`, `status`, `open`) |
| **Functions & Methods** | `camelCase` / `snake_case` starting with an **imperative verb** | `calculateTaxBreakdown()`, `fetchPractitionerSchedule()`, `validatePostalCode()`, `parseWebhookSignature()`, `formatCurrencyMinorUnits()` | Noun-only function names (`taxBreakdown()`, `user()`, `webhook()`) or vague verbs (`handleData()`, `processStuff()`) |
| **Event Handlers & Callbacks** | Props/Contracts: `on<Event>`<br>Internal Handlers: `handle<Subject><Event>` | Prop: `onDosageChange`, `onCheckoutSubmit`<br>Handler: `handleDosageChange`, `handleCheckoutSubmit` | Mixing `onClick` for internal state methods or `clickHandler2` |
| **Classes, Interfaces, Type Aliases & Components** | `PascalCase` (Noun or Noun Phrase) | `LedgerTransaction`, `PatientIntakeForm`, `ConstituencyBoundary`, `SubscriptionPlan` | **Strictly ban** Hungarian `I` prefixes (`IUser`, `IPaymentService`) and redundant `Type` suffixes (`UserType`, `OrderData`) |
| **Generic Type Parameters** | `T` + descriptive `PascalCase` noun (or single `T` only in trivial 1-param containers) | `TEntity`, `TRequestPayload`, `TKey extends PropertyKey`, `TValidationError` | Sequences of cryptic single letters (`<T, U, V, K, P>`) in complex functions |
| **Module-Level Immutable Constants** | `UPPER_SNAKE_CASE` | `MAX_WEBHOOK_RETRY_ATTEMPTS = 5`, `DEFAULT_PAGE_SIZE = 25`, `IDEMPOTENCY_TTL_SECONDS = 86_400` | Naming mutable runtime objects or non-constant environment reads in `UPPER_SNAKE_CASE` |
| **Enums & Discriminated Union Tags** | Type: `PascalCase` (singular)<br>Members/Literals: `UPPER_SNAKE_CASE` or lowercase `'kebab-case'` / `'snake_case'` string literals | Type: `SettlementStatus`<br>Literals: `'pending_review' \| 'settled' \| 'reversed'` | Plural enum names (`SettlementStatuses`) or numeric TypeScript `enum` (which allows unsafe reverse-mapping numbers) |

### 2.2 Mandatory Architectural File Suffixes
File names must reveal their architectural layer at a glance in fuzzy file finders (`Ctrl+P` / `Cmd+P`):
- **UI & Presentation**: `*.component.tsx` / `*.component.ts` / `*.page.tsx` / `*.layout.tsx`
- **State & Reactive Hooks**: `use-*.ts` (React hooks), `*.store.ts`, `*.state.ts`
- **Application & Domain Logic**: `*.service.ts`, `*.use-case.ts`, `*.policy.ts`, `*.mapper.ts`
- **API & Transport Boundary**: `*.controller.ts`, `*.router.ts`, `*.middleware.ts`, `*.guard.ts`
- **Persistence & Data Access**: `*.repository.ts`, `*.entity.ts`, `*.migration.ts`, `*.seed.ts`
- **Contracts & Validation**: `*.schema.ts` (Zod/Valibot), `*.types.ts`, `*.dto.ts`, `*.constants.ts`
- **Verification**: `*.spec.ts` / `*.test.ts` (unit/integration), `*.e2e.ts` (Playwright E2E), `*.fixture.ts`

### 2.3 Explicit Physical, Temporal & Financial Unit Suffixes
Ambiguous numeric and string variables cause catastrophic production incidents (e.g., passing milliseconds where seconds are expected, or dollars where cents are expected). **Always append the unit to the identifier name**:

| Domain Dimension | Mandatory Suffix Standard | Good Identifier Examples | Bad (Ambiguous) Examples |
| :--- | :--- | :--- | :--- |
| **Time & Duration** | `Ms`, `Seconds`, `Minutes`, `Hours`, `Days` | `requestTimeoutMs = 5_000`, `sessionTtlSeconds = 3_600`, `gracePeriodDays = 14` | `timeout = 5000`, `ttl = 3600`, `delay = 60` |
| **Timestamps** | `AtIso`, `AtEpochMs`, `AtUtc` (or typed `Date` / `Instant`) | `publishedAtIso`, `tokenExpiresAtEpochMs`, `recordedAtUtc` | `time`, `date`, `timestamp`, `created` |
| **Monetary Amounts** | `MinorUnits`, `Cents`, `Pence`, `BasisPoints` (`Bps`) | `invoiceTotalMinorUnits`, `unitPriceCents`, `platformFeeBps` (e.g., `150` = 1.50%) | `price`, `amount`, `fee`, `total` (floating-point ambiguity) |
| **Data & Memory Size** | `Bytes`, `Kilobytes`, `Megabytes` | `maxUploadSizeBytes = 10 * 1024 * 1024`, `payloadLengthBytes` | `maxUploadSize`, `fileSize` |
| **Geospatial & Physical** | `Meters`, `Kilometers`, `Degrees`, `Mg` (milligrams), `Bpm` | `searchRadiusMeters`, `dosageMg`, `heartRateBpm`, `latitudeDegrees` | `radius`, `dosage`, `distance` |
| **Ratios & Percentages** | `Ratio` (`0.0–1.0`) vs `Percent` (`0–100`) | `turnoutRatio = 0.642`, `discountPercent = 15` | `rate`, `discount`, `turnout` |

---

## 3. File Internal Ordering & Import Organization

### 3.1 Five-Tier Import Grouping & Sorting
All imports at the top of a file must be organized into **5 distinct groups**, separated by **exactly one blank line**, and **alphabetized by module path** within each group:

1. **Tier 1 — Built-in Runtime Modules**: Always use the explicit `node:` (or `bun:` / `deno:`) protocol prefix (`node:crypto`, `node:fs/promises`, `node:path`).
2. **Tier 2 — External Third-Party Packages**: Framework and npm/registry packages (`@angular/core`, ` fastify`, `react`, `zod`).
3. **Tier 3 — Internal Workspace / Monorepo Aliases**: Path-aliased modules across architectural boundaries (`@core/*`, `@domain/*`, `@shared/*`, `@ui/*`).
4. **Tier 4 — Relative Local Imports**: Sibling or child modules within the same feature slice (`./payment-hash.utils`, `./stripe-webhook.mapper`).
5. **Tier 5 — Type-Only Imports**: Explicit `import type { ... }` declarations (or inline `type` specifiers when enforced by `verbatimModuleSyntax`).

```typescript
// 1. Built-in runtime modules (explicit `node:` prefix)
import { createHmac, timingSafeEqual } from 'node:crypto';

// 2. External third-party packages (alphabetized)
import { z } from 'zod';

// 3. Internal workspace aliases (alphabetized)
import { AppError, ErrorCode } from '@shared/errors/app-error';
import { verifyTenantAccess } from '@shared/security/tenant-guard';

// 4. Relative feature-local imports (alphabetized)
import { mapDisputeToAuditEvent } from './dispute-audit.mapper';
import { DISPUTE_SLA_HOURS } from './dispute.constants';

// 5. Type-only imports (erased completely at compile time)
import type { DisputeRecord, WebhookHeaders } from './dispute.types';
```

### 3.2 Module Boundary & Barrel File Rules
- **Ban Deep Relative Parent Climbs**: Never write `import { x } from '../../../shared/utils/date'`. Any import crossing outside the current feature slice must use a configured workspace path alias (`@shared/utils/date`).
- **Ban Circular Dependencies**: Module `A` importing Module `B` while Module `B` (directly or transitively) imports Module `A` is a `[blocking]` defect (`import/no-cycle`). Extract shared types/contracts into a third leaf module (`*.types.ts`).
- **Avoid Mega-Barrel (`index.ts`) Files in Application Code**: Barrel files that re-export dozens of components and services (`export * from './...'`) defeat bundler tree-shaking, slow down TypeScript language server startup, and introduce circular initialization bugs. Import directly from the specific module file (`./patient-vitals-card.component`) except at the public boundary of a published library package.

### 3.3 Internal File Anatomy (Top-to-Bottom Order)
Within any source module, organize declarations in a predictable top-to-bottom sequence:
1. Module-level file docblock (if architectural context is non-obvious)
2. Five-tier `import` declarations
3. Module-level `UPPER_SNAKE_CASE` constants and compiled schemas/regexes
4. Public exported `interface` / `type` contracts
5. Primary exported class, component, route handler, or service function
6. Private pure helper functions (placed at the bottom or extracted to `*.utils.ts`)

---

## 4. Complexity & Size Budgets

Code that fits inside human working memory is dramatically less likely to harbor security flaws or edge-case regressions. Enforce these quantitative ceilings via static analysis (`eslint`, `sonarjs`, `ruff`, `gocyclo`):

| Metric | Hard Ceiling | Remediation Pattern When Exceeded |
| :--- | :--- | :--- |
| **Cyclomatic Complexity** | **`<= 10`** per function | Replace sprawling `switch` / `if-else` chains with a declarative strategy map (`Record<Status, Handler>`), polymorphism, or smaller domain predicate functions. |
| **Cognitive Complexity** | **`<= 15`** per function | Flatten control flow using early-return guard clauses; extract nested loop bodies into named helper functions. |
| **Maximum Nesting Depth** | **`<= 3`** levels (`if` / `for` / `try` / callback) | Invert conditions and return early (`if (!isValid) { return ...; }`). Never nest `if` inside `if` inside `for` inside `try`. |
| **Function / Component Length** | **`<= 60` lines** of executable code | Extract cohesive sub-steps (validation, transformation, persistence) into single-responsibility helpers or child sub-components. |
| **Source File Length** | **`<= 350` lines** (excluding generated code) | Split god-classes and monolith files by responsibility (`*.service.ts`, `*.mapper.ts`, `*.schema.ts`, `*.types.ts`). |
| **Function Parameters** | **`<= 3` positional parameters** | When a function requires 4+ inputs — or 2+ inputs of the same primitive type (`(string, string, boolean)`) — refactor to a **single typed options/command object** (`createAppointment(params: CreateAppointmentParams)`). |
| **Boolean Flag Parameters** | **0 boolean trap parameters** | Ban calls like `fetchInvoices(tenantId, true, false)`. Replace with an explicit options object (`fetchInvoices(tenantId, { includeArchived: true, hydrateLineItems: false })`) or split into two focused functions. |

```typescript
// ❌ ANTI-PATTERN: Deep nesting (depth 4) + positional boolean trap parameters
function scheduleTelehealthVisit(
  patientId: string,
  practitionerId: string,
  slotIso: string,
  isUrgent: boolean,
  sendSmsNotification: boolean,
): VisitResult {
  if (patientId && practitionerId) {
    if (isSlotAvailable(practitionerId, slotIso)) {
      if (hasInsuranceCoverage(patientId)) {
        // ... deeply nested happy path
      }
    }
  }
}

// ✅ STAFF PATTERN: Typed parameter object + flat guard clauses (nesting depth 1)
interface ScheduleTelehealthVisitParams {
  readonly patientId: PatientId;
  readonly practitionerId: PractitionerId;
  readonly slotStartAtIso: string;
  readonly priority: 'routine' | 'urgent';
  readonly notificationChannel: 'sms' | 'email' | 'none';
}

function scheduleTelehealthVisit(params: ScheduleTelehealthVisitParams): VisitResult {
  if (!isSlotAvailable(params.practitionerId, params.slotStartAtIso)) {
    return { ok: false, reason: 'SLOT_UNAVAILABLE' };
  }
  if (!hasInsuranceCoverage(params.patientId)) {
    return { ok: false, reason: 'INSURANCE_UNVERIFIED' };
  }

  return bookConfirmedVisit(params);
}
```
