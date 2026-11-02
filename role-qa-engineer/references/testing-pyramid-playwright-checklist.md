# Testing Pyramid & Playwright Automation Checklist

> **Baseline Tooling & Standard Versions (Rule G)**
> - **End-to-End & Component Browser Testing**: Playwright `1.49+` (`@playwright/test`)
> - **Unit & Integration Test Runner**: Vitest `2.x` / Jest `29.7+`
> - **DOM & User Interaction Utilities**: Testing Library (`@testing-library/dom` `10.x`, `@testing-library/user-event` `14.x`)
> - **Automated Accessibility Auditing**: `@axe-core/playwright` `4.10+` (WCAG 2.2 Level A & AA)
> - **Network Interception & API Mocking**: Mock Service Worker (`msw` `2.6+`)
> - **Database Isolation**: Testcontainers (`@testcontainers/postgresql` `10.x`) / Transactional Rollbacks
> - **Performance & Load Testing**: Grafana k6 `0.54+`

---

## 1. Locator Strategy & Semantic Priority

Playwright and Testing Library locators must mirror how assistive technologies and real users perceive the interface.

### Strict Locator Priority Order
1. **`getByRole` (Highest Priority)**: Queries elements by ARIA role and accessible name. Simultaneously verifies that the UI is accessible to screen readers.
2. **`getByLabel`**: Queries form controls (`<input>`, `<select>`, `<textarea>`) associated with a `<label>` or `aria-labelledby`.
3. **`getByPlaceholder`**: Use only when a visible label is absent (though forms should always have visible labels).
4. **`getByText`**: Queries non-interactive text containers (headings, paragraphs, status badges, toast messages).
5. **`getByAltText` / `getByTitle`**: Queries images (`<img>`) and SVGs with accessible descriptions.
6. **`getByTestId` (`data-testid`)**: Escape hatch only when semantic roles or text are dynamic or non-unique (e.g., canvas charts, geospatial map markers, skeleton placeholders).

### Anti-Patterns vs. Staff-Grade Patterns
```typescript
// ❌ ANTI-PATTERN: Fragile CSS/XPath tied to styling and DOM nesting
await page.locator('div.card-wrapper > form > button.bg-blue-600:nth-child(2)').click();
await page.locator('//table/tbody/tr[1]/td[4]/a').click();

// ✅ STAFF PATTERN: Resilient, accessibility-validating user locators
// E-commerce / FinTech example:
await page.getByRole('button', { name: /authorize payment/i }).click();

// Healthcare / SaaS scoped row action example:
const patientRow = page.getByRole('row', { name: /elena rodriguez/i });
await patientRow.getByRole('link', { name: /view lab results/i }).click();
```

### Checklist — Locators
- [ ] Zero XPath selectors (`//div...`) or Tailwind utility class selectors (`.flex.items-center...`) in test files.
- [ ] Chained and filtered locators (`locator.filter({ hasText })` or `within(row)`) are used to disambiguate repeated items in lists, tables, or cards.
- [ ] Every interactive element that fails `getByRole(..., { name })` lookup triggers a fix to the underlying HTML/ARIA markup rather than slapping a `data-testid` on an inaccessible `<div onClick>`.

---

## 2. Eliminating Flaky Tests (Determinism & Web-First Assertions)

Flaky tests destroy engineering trust in CI. Almost all flakiness stems from race conditions between test execution, browser rendering, animations, and asynchronous I/O.

### Rules for 100% Deterministic Tests
1. **Never Use Hardcoded Sleeps**:
   - Ban `await page.waitForTimeout(2000)` and `new Promise(r => setTimeout(r, 1000))`.
   - Enforce ESLint rule `playwright/no-wait-for-timeout: "error"`.
2. **Always Use Web-First Auto-Retrying Assertions**:
   - Web-first assertions poll the DOM automatically until the condition is met or the timeout expires.
3. **Synchronize on Network or UI State Transitions, Not Guesswork**:
   - Wait for specific responses (`const responsePromise = page.waitForResponse(...)` registered *before* triggering the click) or assert the resulting UI state directly (`await expect(page.getByRole('status')).toHaveText(/saved/i)`).
4. **Deterministic Clock Control (`page.clock` / `vi.useFakeTimers`)**:
   - Use Playwright's `page.clock.install({ time: new Date('2026-11-01T01:30:00Z') })` to test token expiration, scheduled publishing, relative timestamps (`"3 minutes ago"`), and Daylight Saving Time (DST) transitions without waiting in real time.

```typescript
// ❌ ANTI-PATTERN: Non-retrying synchronous read + hardcoded sleep
await page.getByRole('button', { name: /export report/i }).click();
await page.waitForTimeout(3000);
expect(await page.getByRole('alert').isVisible()).toBe(true); // Evaluates once and fails if takes 3050ms!

// ✅ STAFF PATTERN: Web-first auto-retrying assertion
await page.getByRole('button', { name: /export report/i }).click();
await expect(page.getByRole('alert')).toBeVisible();
await expect(page.getByRole('alert')).toContainText(/report ready for download/i);
```

### Checklist — Flakiness Prevention
- [ ] All `expect(...)` calls on Playwright locators are `await`ed (`playwright/missing-playwright-await: "error"`).
- [ ] No race conditions between `click()` and hydration: pages wait for interactive readiness before firing input events.
- [ ] New E2E tests pass `npx playwright test <spec> --repeat-each=5 --workers=4` locally and in CI before merge.

---

## 3. Page Object Model (POM), Fixtures & Auth State Reuse

### Modular Page Objects & Custom Fixtures
- Encapsulate complex screen interactions into composable **Page Objects** or **Component Drivers** while keeping assertions visible in the test body (or dedicated `expect*` methods).
- Use **Playwright Custom Fixtures (`test.extend`)** instead of `beforeEach`/`afterEach` globals for dependency injection, automatic cleanup, and parallel worker isolation.
- **Fast Authentication via `storageState`**:
  - Test the actual login UI flow in a dedicated `auth.spec.ts`.
  - For all other 95% of tests, authenticate once in a setup project (`auth.setup.ts`) via API or browser, save cookies/localStorage to `playwright/.auth/user.json`, and inject via `use: { storageState }`.

```typescript
// tests/fixtures/workspace.fixture.ts
import { test as base, expect } from '@playwright/test';
import { CheckoutPage } from '../pages/checkout.page';

type WorkspaceFixtures = {
  checkoutPage: CheckoutPage;
  seededOrderId: string;
};

export const test = base.extend<WorkspaceFixtures>({
  seededOrderId: async ({ request }, use) => {
    // Setup isolated test data via fast internal test API
    const res = await request.post('/api/test-utils/seed-order', {
      data: { currency: 'EUR', items: [{ sku: 'MED-KIT-01', qty: 2 }] },
    });
    const { orderId } = await res.json();
    await use(orderId);
    // Teardown automatically after test completes (even on failure)
    await request.delete(`/api/test-utils/seed-order/${orderId}`);
  },
  checkoutPage: async ({ page, seededOrderId }, use) => {
    const checkout = new CheckoutPage(page, seededOrderId);
    await checkout.goto();
    await use(checkout);
  },
});

export { expect };
```

---

## 4. Network Interception, MSW 2.x & Contract Testing

### When to Mock vs. When to Hit Real Backend
- **E2E Happy Paths**: Hit the real application backend and real test database. Only mock external third-party APIs outside your control (e.g., Stripe webhooks, national identity providers, external geocoding APIs, SMS gateways).
- **Edge Cases & Error States (500s, 429s, Timeouts)**: Use `page.route()` in Playwright or `server.use(http.get(...))` in MSW 2.x to deterministically simulate network failures, rate limiting (`429 Too Many Requests` with `Retry-After`), and malformed payloads.
- **Schema Contract Verification**: Validate that mocked responses in MSW/Playwright strictly conform to the shared Zod/OpenAPI schema so mocks never drift from real backend contracts.

```typescript
// Simulating a 503 Service Unavailable with retry recovery in Playwright
test('gracefully displays retry banner when search API returns 503 and recovers on retry', async ({ page }) => {
  let attempts = 0;
  await page.route('**/api/v1/search*', async (route) => {
    attempts += 1;
    if (attempts === 1) {
      await route.fulfill({
        status: 503,
        contentType: 'application/problem+json',
        body: JSON.stringify({ title: 'Service Temporarily Unavailable', status: 503 }),
      });
    } else {
      await route.continue();
    }
  });

  await page.goto('/directory');
  await page.getByRole('searchbox', { name: /search providers/i }).fill('Cardiology');
  await expect(page.getByRole('alert')).toContainText(/temporarily unavailable/i);

  await page.getByRole('button', { name: /try again/i }).click();
  await expect(page.getByRole('heading', { name: /cardiology specialists/i })).toBeVisible();
});
```

---

## 5. Automated Accessibility (`@axe-core/playwright`) & Visual Regression

### Automated WCAG 2.2 AA Scanning
Every route and interactive state (modals, drawers, dropdowns, error summaries, dark mode) must be audited with `@axe-core/playwright`.

```typescript
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('profile overview and edit modal meet WCAG 2.2 AA accessibility standards', async ({ page }) => {
  await page.goto('/profiles/dr-sarah-chen');
  await expect(page.getByRole('heading', { level: 1 })).toBeVisible();

  const initialScan = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa', 'wcag22aa'])
    .analyze();
  expect(initialScan.violations).toEqual([]);

  // Open interactive modal/drawer and re-scan
  await page.getByRole('button', { name: /suggest correction/i }).click();
  await expect(page.getByRole('dialog')).toBeVisible();

  const dialogScan = await new AxeBuilder({ page })
    .include('[role="dialog"]')
    .withTags(['wcag2a', 'wcag2aa', 'wcag22aa'])
    .analyze();
  expect(dialogScan.violations).toEqual([]);
});
```

### Visual Regression (`toHaveScreenshot`)
- Disable animations (`animations: 'disabled'`) and mask volatile dynamic regions (live timestamps, avatars, third-party map tiles) via `mask: [page.locator('[data-visual-mask]')]`.
- Run visual snapshot comparisons inside a pinned container environment (e.g., official `mcr.microsoft.com/playwright:v1.49.0-noble` Docker image) so font rendering is bit-for-bit identical across developer machines and CI.

---

## 6. Mobile Viewport, Touch Ergonomics & Responsive Matrix

Configure `playwright.config.ts` to test across desktop and mobile device descriptors:
- **Mobile iOS**: `devices['iPhone 15']` (`390x844`, `isMobile: true`, `hasTouch: true`, WebKit)
- **Mobile Android**: `devices['Pixel 7']` (`412x915`, `isMobile: true`, `hasTouch: true`, Chromium)
- **Tablet**: `devices['iPad Air']` (`820x1180`, `hasTouch: true`)
- **Desktop**: `1280x800` / `1440x900` (Chromium, Firefox, WebKit)

### Mobile-Specific QA Verification Checklist
- [ ] **No Horizontal Overflow**: Verify `document.documentElement.scrollWidth <= document.documentElement.clientWidth` at `320px` and `390px` widths.
- [ ] **Touch Target Geometry (`>= 48x48px`)**: Assert bounding box dimensions (`const box = await locator.boundingBox()`) on primary interactive controls and navigation buttons.
- [ ] **Sticky Bar & Safe-Area Collision**: Verify bottom sticky CTA bars or cookie banners do not obscure form submit buttons or last list items when scrolled to the bottom.
- [ ] **iOS Auto-Zoom Prevention**: Verify `<input>`, `<select>`, and `<textarea>` elements compute to `font-size >= 16px` on mobile viewports so iOS Safari does not forcefully zoom the page on focus.

---

## 7. Database & Integration Test Isolation

- **Never Share Mutable DB Rows Across Parallel Workers**:
  - **Pattern A (Transaction Rollback)**: Wrap each integration test in a database transaction (`BEGIN` -> run test -> `ROLLBACK`) when testing repository/service layers in-process.
  - **Pattern B (Per-Worker Schema / Database or Scoped Tenant IDs)**: In Playwright E2E tests where the server runs in a separate process, namespace test entities using unique tenant/workspace IDs per test (`crypto.randomUUID()`) or spin up per-worker schemas via `TEST_PARALLEL_INDEX`.
- **Migration Verification**: CI must run `migrate:up` from an empty database and run seed scripts before executing integration/E2E suites.

---

## 8. Boundary, Mutation & Chaos Edge-Case Matrix

Senior QA engineers actively try to break the system using adversarial inputs and environmental stress:

| Category | Scenarios to Test | Expected System Behavior |
| :--- | :--- | :--- |
| **Network Chaos** | Slow 3G throttling, abrupt offline drop (`context.setOffline(true)`) mid-form submission, high latency (2,500ms). | Shows loading/skeleton state, preserves user's typed form data locally, shows clear offline/retry message without crashing. |
| **Concurrency & Idempotency** | Rapid double/triple click on "Submit Order" / "Cast Vote" / "Book Appointment" button within 50ms. | Button disables immediately (`aria-busy="true"`), only 1 network mutation fires, and backend idempotency key prevents duplicate records. |
| **Boundary Values** | `0`, `-1`, `Number.MAX_SAFE_INTEGER`, floating-point precision (`0.1 + 0.2`), empty arrays `[]`, pagination page `0` or `999999`. | Exact validation error (`400`/`422`) or accurate bounded calculation without `NaN` or `500 Internal Server Error`. |
| **Unicode, i18n & Overflow** | Multi-byte emojis (`👩‍⚕️🏴󠁧󠁢󠁳󠁣󠁴󠁿`), RTL scripts (Arabic/Hebrew), diacritics (`Łódź`, `Zürich`, `São Paulo`), 500-char unbroken strings, SQL/XSS metacharacters (`'"><script>`). | UTF-8 preserved end-to-end, layout truncates/wraps cleanly without breaking grid, zero script execution. |
| **Temporal & Timezone** | User in `UTC-12`, `UTC+14`, or crossing DST boundary (`01:59` -> `03:00`); leap day (`Feb 29`); expired JWT mid-session. | Server stores UTC ISO-8601; UI renders local timezone accurately; expired session refreshes transparently or redirects to login preserving return URL. |
| **Browser History & Deep Links** | Refreshing (`F5`) mid-wizard, clicking Browser Back/Forward after modal open or filter selection, sharing filtered URL in incognito. | URL query/path state restores exact filters/step; Back button closes modal or returns to previous state cleanly. |
