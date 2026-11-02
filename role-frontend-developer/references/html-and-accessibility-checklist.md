# Semantic HTML, Form Ergonomics & WCAG 2.2 AA Accessibility Checklist (`references/html-and-accessibility-checklist.md`)

> **Baseline Versions**: HTML Living Standard (2025/2026), WAI-ARIA `1.2` / `1.3`, WAI-ARIA Authoring Practices Guide (APG), WCAG `2.2 Level AA`.
> **Mandate**: Load and enforce this checklist whenever authoring, refactoring, or reviewing HTML templates, interactive widgets, modals/popovers, forms, or accessibility compliance.

> [!NOTE]
> **Continuous Evolution Mandate**: This checklist is a baseline quality floor, never a ceiling. Engineers must proactively verify real keyboard navigation, screen-reader announcement fidelity, focus management across asynchronous route/view transitions, and emerging ARIA/HTML capabilities.

---

## 1. Semantic HTML & High-Conversion Form Ergonomics

### 1.1 Document Landmarks & Semantic Elements
Screen reader users navigate pages via landmark regions and heading hierarchies (`<h1>` through `<h6>`, never skipping levels for visual sizing).
- **Mandatory Landmarks**:
  - `<header>`: Top-level site banner (renders as `role="banner"` when direct child of `<body>`).
  - `<nav aria-label="Primary">` / `<nav aria-label="Breadcrumb">`: Always label `<nav>` elements when more than one exists on the page.
  - `<main id="main-content" tabindex="-1">`: Exactly one visible `<main>` per document, targeted by the first focusable `<a href="#main-content" class="sr-only focus:not-sr-only">Skip to main content</a>` link.
  - `<search>`: Wrap search and filtering forms (e.g., E-commerce product search, Civic ballot lookup, Healthcare provider directory filter).
  - `<aside>` and `<footer>`: Complementary content and document footer.
- **`<button>` vs. `<a href>` Rule**:
  - **Navigation** (changes URL, supports middle-click / Cmd+Click to open in new tab): Always use `<a href="...">`.
  - **Action** (submits form, opens modal, toggles menu, deletes record, sorts table): Always use `<button type="button">` (default `<button>` inside a form is `type="submit"`, so always specify `type="button"` for non-submit buttons!).
  - **Never** attach click handlers to `<div>` or `<span>`.

### 1.2 Native `<dialog>` & Popover API (`popover`)
- **Modal Dialogs (`<dialog>`)**: Prefer the native `<dialog>` element opened via `.showModal()`. It automatically provides top-layer promotion, `Escape` key dismissal, background inertness, focus trapping, and `::backdrop` styling.
- **Non-Modal Popovers (`popover="auto"`)**: Use native HTML `popover` (`<button popovertarget="filter-menu">` + `<div id="filter-menu" popover="auto">`) for lightweight dropdowns, tooltips, and action menus with built-in light-dismiss behavior.

### 1.3 Background Tree Inertness (`inert` Attribute)
- **Native `inert` Attribute**: When opening custom non-`<dialog>` overlays (e.g., custom mobile bottom sheets, slide-over drawers) or when a multi-step checkout/clinical section is locked while saving, apply the boolean `inert` attribute to background or disabled container trees (`<main inert>`).
- **Effect of `inert`**: Removes all interactive descendants from the tab order (`tabindex`), blocks pointer/click events, and hides the subtree from the accessibility tree (`aria-hidden` equivalent) in a single atomic browser primitive—preventing focus from escaping behind an active overlay.
- **Lifecycle Rule**: Always remove `inert` and restore focus to the invoking trigger element immediately when the overlay closes.

### 1.4 Mobile-First Form Ergonomics (`inputmode`, `autocomplete`, `enterkeyhint`)
Every form input must configure mobile virtual keyboards and password/autofill managers accurately:

| Field Use Case (Multi-Domain) | `type` | `inputmode` | `autocomplete` | `enterkeyhint` |
| :--- | :--- | :--- | :--- | :--- |
| 6-Digit SMS / Authenticator OTP | `text` | `numeric` | `one-time-code` | `done` |
| FinTech / E-commerce Price or Amount | `text` | `decimal` | `off` | `next` |
| Credit Card Number | `text` | `numeric` | `cc-number` | `next` |
| Phone Number (Healthcare / Delivery) | `tel` | `tel` | `tel` | `next` |
| Email Address | `email` | `email` | `email` | `next` |
| Search Bar (Catalog / Directory) | `search` | `search` | `off` | `search` |
| Password (Login vs. Registration) | `password` | *(omit)* | `current-password` / `new-password` | `done` |

> [!WARNING]
> **Never use `<input type="number">` for credit cards, postal codes, SSNs/national IDs, or OTP codes.** `type="number"` strips leading zeros, allows scientific notation (`e`), and accidentally increments/decrements values on mouse-wheel scroll. Use `<input type="text" inputmode="numeric" pattern="[0-9]*">` instead.

### 1.5 Responsive Images & Media Performance
- **Above-the-Fold LCP Hero Image**:
  ```html
  <img
    src="/hero-clinic-1200.avif"
    srcset="/hero-clinic-640.avif 640w, /hero-clinic-1200.avif 1200w, /hero-clinic-1920.avif 1920w"
    sizes="(max-width: 768px) 100vw, 50vw"
    width="1200"
    height="800"
    fetchpriority="high"
    loading="eager"
    alt="Patients checking in at the downtown clinic reception desk" />
  ```
- **Below-the-Fold Images**: Always set `loading="lazy"`, `decoding="async"`, and explicit `width` & `height` attributes to reserve aspect ratio and guarantee zero Cumulative Layout Shift (CLS).
- **Decorative Images**: Set `alt=""` (empty string) so screen readers skip them completely. Never omit the `alt` attribute altogether.

---

## 2. Accessibility (WCAG 2.2 Level AA Mandatory Engineering)

### 2.1 The First Rule of ARIA
> **"No ARIA is better than bad ARIA."**
- Never add `role="button"` to a `<button>` or `role="navigation"` to a `<nav>`.
- If you must build a custom composite widget (Combobox, Tabs, Treegrid, Modal), follow the **WAI-ARIA Authoring Practices Guide (APG)** keyboard interaction contract (`ArrowLeft`/`ArrowRight`, `Home`, `End`, `Escape`, roving `tabindex="0"` / `tabindex="-1"`).

### 2.2 Roving `tabindex` & Composite Widget Keyboard Matrix
Composite widgets must act as a **single Tab stop** in the page sequence. Pressing `Tab` moves focus into the widget (onto the currently active item with `tabindex="0"`), Arrow keys move focus *between* items within the widget (updating the active item to `tabindex="0"` and all siblings to `tabindex="-1"`), and pressing `Tab` again moves focus out to the next page control:

| Composite Widget | ARIA Roles & Attributes | Keyboard Navigation Contract (WAI-ARIA APG) |
| :--- | :--- | :--- |
| **Tabs** (e.g., SaaS Account Settings, EHR Patient Chart Sections) | `role="tablist"`, `role="tab"` (`aria-selected="true\|false"`, `aria-controls="panel-id"`, roving `tabindex`), `role="tabpanel"` (`aria-labelledby="tab-id"`). | `Tab`: Enters active tab, then moves to tabpanel.<br>`ArrowLeft` / `ArrowRight` (or `ArrowUp`/`ArrowDown` if vertical): Moves focus across tabs (with wrapping).<br>`Home` / `End`: Jumps to first / last tab. |
| **Action Menu / Dropdown Menu** (e.g., Row actions in FinTech ledger or E-commerce order table) | Trigger `<button aria-haspopup="menu" aria-expanded="true\|false">`, container `role="menu"`, items `role="menuitem"` (`role="menuitemcheckbox"`, `role="menuitemradio"`). | `Enter` / `Space` / `ArrowDown` on trigger: Opens menu & focuses first item.<br>`ArrowDown` / `ArrowUp`: Moves between menu items.<br>`Home` / `End`: First / last item.<br>`Escape`: Closes menu and returns focus to trigger button. |
| **Radio Group / Segmented Control** (e.g., Billing cycle toggle, Civic ballot filter) | `role="radiogroup"` (`aria-labelledby`), items `role="radio"` (`aria-checked="true\|false"`, roving `tabindex="0"` on checked or first item, `-1` on others). | `Tab`: Focuses checked radio item.<br>`ArrowRight` / `ArrowDown`: Moves focus and checks next option.<br>`ArrowLeft` / `ArrowUp`: Moves focus and checks previous option.<br>`Space`: Checks focused item if not already checked. |

```html
<!-- Roving tabindex example: Horizontal Tablist -->
<div role="tablist" aria-label="Order Fulfilment Views">
  <button role="tab" id="tab-active" aria-selected="true" aria-controls="panel-active" tabindex="0" type="button">
    Active Shipments (12)
  </button>
  <button role="tab" id="tab-returns" aria-selected="false" aria-controls="panel-returns" tabindex="-1" type="button">
    Returns (2)
  </button>
</div>
<div role="tabpanel" id="panel-active" aria-labelledby="tab-active" tabindex="0">
  <!-- Active shipments content -->
</div>
```

### 2.3 WCAG 2.2 New Success Criteria Checklist
1. **2.4.11 Focus Not Obscured (Minimum) (AA)**: When any item receives keyboard focus, it must not be entirely hidden behind sticky headers, sticky footers, or floating cookie/chat banners. Use `scroll-padding-top` and `scroll-padding-bottom` on `html` matching your sticky bar heights!
2. **2.5.7 Dragging Movements (AA)**: Any action that uses drag-and-drop (e.g., SaaS Kanban board reordering, E-commerce price range slider, Map pan) **must** provide a single-pointer / keyboard alternative (e.g., Move Up / Move Down buttons, numeric inputs).
3. **2.5.8 Target Size (Minimum) (AA)**: Every pointer target must be at least `24x24 CSS pixels` (or have `24px` spacing offset), and mobile touch controls should target **`44x44px` to `48x48px`**.
4. **3.2.6 Consistent Help (A)**: Help mechanisms (contact link, support chat, FAQ) must appear in the same relative DOM order across pages.
5. **3.3.7 Redundant Entry (A)**: Never force users to re-type information already provided in the same multi-step session (e.g., "Same as shipping address" checkbox in E-commerce checkout).
6. **3.3.8 Accessible Authentication (Minimum) (AA)**: **Never block paste (`onpaste="return false"`)** on password or OTP inputs, and properly support password managers (`autocomplete="current-password"`) and WebAuthn/Passkeys.

### 2.4 Accessible Form Validation & Live Regions
```html
<div class="field">
  <label for="iban-input">Recipient IBAN</label>
  <input
    id="iban-input"
    name="iban"
    type="text"
    aria-required="true"
    aria-invalid="true"
    aria-describedby="iban-hint iban-error" />
  <p id="iban-hint" class="text-muted">Enter the 22-character international bank account number.</p>
  <p id="iban-error" role="alert" class="text-danger">
    Invalid IBAN checksum. Please verify the digits on your statement.
  </p>
</div>
```
- **Asynchronous Status Announcements (`aria-live`)**: Mount a persistent polite live region (`<div aria-live="polite" aria-atomic="true" class="sr-only">`) in the DOM *before* injecting status text (e.g., `"Filter applied: 18 healthcare providers found"`, `"Item added to cart"`). Screen readers frequently ignore `aria-live` containers that are dynamically inserted into the DOM at the exact same moment as their text content.
