# UI Craftsmanship, Visual Polish & Mobile Ergonomics Checklist

> **Baseline Tooling & Standard Versions (Rule G)**
> - **Accessibility Standards**: W3C WCAG `2.2 Level AA` + APCA (Accessible Perceptual Contrast Algorithm) guidelines
> - **Visual Design & Prototyping**: Penpot `2.x` (Native CSS Grid, Flex Layout, Design Tokens)
> - **Styling & Token Engine**: Tailwind CSS `4.0` (`@theme` CSS-first configuration) / Tailwind CSS `3.4+` / Modern Baseline CSS
> - **Modern Browser APIs**: CSS Container Queries (`@container`), CSS `:has()`, CSS View Transitions API (`document.startViewTransition`), Popover API & Native `<dialog>`

---

## 1. Typography & Optical Hierarchy

Typography is the primary structure of any interface. Poor typographic defaults immediately expose amateur or uncurated AI output.

### Fluid Type Scale & Measure
- **Fluid Sizing with `clamp()`**: Scale display and heading sizes smoothly between `390px` mobile and `1440px` desktop viewports without abrupt breakpoint jumps, while keeping body copy anchored at a legible `1rem` (`16px`).
- **Line Width (Measure)**: Constrain long-form reading paragraphs (articles, legal disclosures, clinical notes, documentation) to **`60ch–72ch`** (`max-w-prose`). Never allow body text to stretch edge-to-edge across a `1440px` monitor.
- **Proportional Line Height & Letter Spacing**:
  - Display / `H1` (`clamp(1.875rem, 3vw + 1rem, 3rem)`): `line-height: 1.1–1.15`, `letter-spacing: -0.025em`.
  - Section Headings `H2`–`H3`: `line-height: 1.2–1.3`, `letter-spacing: -0.015em`.
  - Body Copy (`1rem` / `16px`): `line-height: 1.5–1.65`, `letter-spacing: normal`.
  - All-Caps Eyebrows / Badges (`0.75rem` / `12px`): `letter-spacing: 0.06em`, `font-weight: 600`.

### Wrapping, Numeric Alignment & Optical Corrections
- **`text-wrap: balance`**: Apply to all headings, modal titles, card headers, and centered callouts so multi-line headings never leave a single dangling word on the final line.
- **`text-wrap: pretty`**: Apply to body copy and descriptions to optimize hyphenation and prevent orphans at the end of paragraphs.
- **`font-variant-numeric: tabular-nums`**: Mandatory on all financial amounts, stock tickers, clinical lab readings, election percentages, timestamps, countdowns, pagination counters, and data table cells.
- **`font-variant-numeric: slashed-zero`**: Use in reference codes, order IDs, serial numbers, and cryptographic hashes to disambiguate `0` from `O`.
- **Optical Alignment**: Icons paired with text inside buttons (e.g., a left-arrow + `"Back"` or `"Continue"` + right-arrow) often require `2px` optical padding adjustment (`pl-3.5 pr-4`) so visual weight feels centered.

```css
/* Tailwind CSS 4.0 / Modern CSS Typographic Baseline */
@layer base {
  h1, h2, h3, h4, [data-balance] {
    text-wrap: balance;
    letter-spacing: -0.02em;
  }
  p, dd, li {
    text-wrap: pretty;
  }
  .numeric, table td[data-type="number"], time, [data-tabular] {
    font-variant-numeric: tabular-nums;
  }
}
```

### Checklist — Typography
- [ ] Headings use `text-wrap: balance`; body paragraphs use `text-wrap: pretty` and `max-w-prose` (`60ch–72ch`).
- [ ] All numbers in tables, prices, counters, and badges use `tabular-nums`.
- [ ] Font loading uses `font-display: swap` or `optional` with size-adjusted fallback metrics (`size-adjust`, `ascent-override`) to eliminate layout shift (FOUT/CLS).
- [ ] Zero text below `12px` (`0.75rem`) anywhere in the UI; mobile form `<input>` text is `>= 16px` (`1rem`).

---

## 2. Color Systems, OKLCH & Dark Mode Surface Elevation

### Perceptually Uniform Color Spaces (`OKLCH`)
- Prefer **OKLCH (`oklch(L C H)`)** over legacy RGB/HSL when defining color ramps in CSS/Tailwind 4.0. In OKLCH, equal steps in Lightness (`L`) produce identical perceived contrast across all hues (preventing the classic HSL bug where yellow looks blindingly bright and blue looks muddy at the same `50%` lightness).
- **Semantic Token Indirection**: Components must consume semantic CSS variables (`--surface-card`, `--text-primary`, `--border-subtle`), never raw palette steps (`bg-gray-100 dark:bg-gray-800` scattered across 200 files).

### Dark Mode Done Right (Surface Luminance vs. Shadows)
- **Never Pure `#000000` Canvas with Pure `#FFFFFF` Text**: High-halation pure white on pure black causes visual vibration (astigmatism halation). Use rich dark neutral canvas surfaces (e.g., `oklch(0.14 0.01 260)`) and slightly softened primary text (`oklch(0.95 0.005 260)`).
- **Elevation via Surface Lightness, Not Dark Drop Shadows**:
  - In Light Mode, elevation is communicated via subtle ambient + key drop shadows and white cards on a warm/cool off-white canvas (`oklch(0.985 0.003 260)`).
  - In Dark Mode, black drop shadows are invisible against dark backgrounds. Communicate elevation by making higher surfaces progressively **lighter** (`Canvas L=0.14` -> `Card L=0.18` -> `Dropdown/Modal L=0.22`) paired with a crisp `1px solid oklch(1 0 0 / 0.08)` inner/outer border highlight.
- **Status Colors (Success, Warning, Danger, Info)**:
  - Desaturate and adjust lightness of semantic status badges in Dark Mode so error/warning banners remain WCAG 2.2 AA readable without burning into the screen.
  - **Never rely on color alone (WCAG 1.4.1)**: Every status indicator must pair color with a distinct icon (`CheckCircle`, `AlertTriangle`, `XOctagon`) or explicit text label.

### Checklist — Color & Contrast
- [ ] Normal text achieves `>= 4.5:1` contrast (APCA `Lc >= 75` for body copy); large text and interactive borders/icons achieve `>= 3.0:1` (APCA `Lc >= 60`).
- [ ] Dark mode elevates cards/modals using progressive surface lightness + `1px` translucent borders (`white/8%`) rather than invisible dark shadows.
- [ ] All semantic states (Error, Warning, Success, Active Filter) remain distinguishable under deuteranopia/protanopia/achromatopsia color-blindness simulation.

---

## 3. Layout, Spatial Rhythm & Mobile Geometry

### 4px / 8px Spatial Grid & Internal vs. External Spacing
- **Law of Proximity**: The outer spacing *between* distinct groups/cards (`gap-6` / `24px`) must always be noticeably larger than the inner padding (`p-4` / `16px`) and internal element spacing (`gap-2` / `8px`) *within* a card.
- **Concentric Border Radii**: When nesting a rounded element (e.g., an image or button with inner radius $r_{inner}$) inside a padded container with padding $p$ and outer radius $r_{outer}$, calculate concentric curvature:
  $$r_{outer} = r_{inner} + p$$
  *(Example: An inner thumbnail with `rounded-lg` (`8px`) inside a card with `p-2` (`8px`) padding requires an outer card radius of `rounded-2xl` (`16px`) to look optically parallel.)*

### Container Queries & Safe-Area Insets
- **Component-Level Responsiveness (`@container`)**: Use CSS Container Queries (`@container` / Tailwind `@md:flex-row`) for reusable cards, widgets, and data panels so they adapt to their parent column width (sidebar vs. main canvas) rather than only the global `window` viewport.
- **Mobile Safe Areas & Dynamic Viewport Units**:
  - Use `min-h-dvh` (`100dvh`) instead of `100vh` so mobile browser address bars do not clip bottom content.
  - Pad sticky bottom bars and drawers with `padding-bottom: max(1rem, env(safe-area-inset-bottom))`.

### Checklist — Layout & Geometry
- [ ] Zero horizontal scrollbar (`overflow-x`) at `320px`, `375px`, `390px`, `768px`, `1024px`, and `1440px`.
- [ ] All interactive controls meet a minimum `48x48px` hit target on touch devices (using invisible hit-area expansion via `relative after:absolute after:-inset-2` when the visual icon is smaller).
- [ ] Nested rounded corners obey concentric radius geometry ($r_{outer} = r_{inner} + p$).
- [ ] Sticky headers/footers use `dvh` and `env(safe-area-inset-*)` and never obscure focused elements when tabbing (`scroll-margin-top` / `scroll-padding-top`).

---

## 4. Micro-Interactions, Motion & Loading Etiquette

### Compositor-Only Motion & View Transitions
- **Allowed Animated Properties**: `transform` (`translate3d`, `scale`, `rotate`), `opacity`, and `filter` (sparingly).
- **View Transitions API**: Use `document.startViewTransition()` (with graceful fallback) for shared-element transitions between list cards and detail headers or tab switches.
- **Popover & Dialog Physics**:
  - Scale modals/popovers from `scale(0.96)` and `opacity: 0` to `scale(1)` and `opacity: 1` from their trigger origin (`transform-origin: var(--radix-popover-content-transform-origin)`), never from `scale(0)` (which looks cartoonish).

### Skeleton Shimmer vs. Spinner Etiquette
- **< 150ms response**: Show nothing (instant transition; flashing a spinner for 80ms makes the app feel slower and jittery).
- **150ms – 300ms**: Show subtle button busy state or progress bar.
- **> 300ms initial page/section load**: Render **geometry-accurate Skeleton placeholders** with a calm shimmer (`prefers-reduced-motion: reduce` switches shimmer to static muted pulse).
- **Optimistic UI**: For deterministic, low-risk actions (bookmarking an item, toggling a filter, reordering a task), update the UI immediately and roll back with a toast notification only if the background request fails.

### Checklist — Motion & Feedback
- [ ] Every interactive element has distinct `:hover`, `:focus-visible`, `:active` (pressed), and `:disabled` visual states.
- [ ] Zero layout-thrashing animations (`height`, `width`, `top`, `left`, `margin`).
- [ ] `@media (prefers-reduced-motion: reduce)` disables spatial movement, parallax, and infinite shimmers/spinners.

---

## 5. Form & Input Ergonomics

Forms are where user friction and drop-off are highest.

### Validation Timing: "Reward Early, Punish Late"
1. **While typing in a virgin field (Initial Entry)**: Do **NOT** scream red error messages while the user is only on the 3rd character of typing their email address (`"Invalid email!"` on `"joh"` is hostile UX).
2. **On Blur (Leaving the field) or Submit**: Validate when the user leaves the field (`onBlur`) or attempts submission (`onSubmit`) — *punish late*.
3. **While fixing an existing error (Re-validation)**: Once a field is already marked invalid, validate on every keystroke (`onChange`) and clear the error state the exact millisecond the input becomes valid — *reward early*.

### Mobile Input Attributes & Accessibility Plumbing
- Always specify semantic HTML input attributes so mobile OS keyboards display the optimal key layout and autofill bar:
  - Email: `type="email" inputmode="email" autocomplete="email" autocapitalize="none" spellcheck="false"`
  - Phone: `type="tel" inputmode="tel" autocomplete="tel"`
  - OTP / Verification Code: `type="text" inputmode="numeric" pattern="[0-9]*" autocomplete="one-time-code"`
  - Currency / Decimal: `type="text" inputmode="decimal"` (avoid `type="number"` for credit cards, postal codes, or currency because `type="number"` strips leading zeros and accidentally increments/decrements values on mouse wheel scroll!).
- **Programmatic Error Linking**:
  ```html
  <div class="field-group">
    <label for="national-id" class="field-label">National Identification Number</label>
    <p id="national-id-hint" class="field-hint">Enter your 11-digit civic ID without spaces.</p>
    <input
      id="national-id"
      name="nationalId"
      type="text"
      inputmode="numeric"
      aria-invalid="true"
      aria-describedby="national-id-hint national-id-error"
    />
    <p id="national-id-error" role="alert" class="field-error">
      National ID must contain exactly 11 digits (you entered 9).
    </p>
  </div>
  ```

### Checklist — Form Ergonomics
- [ ] Labels are permanently visible above inputs (never rely solely on disappearing `placeholder` text as a label).
- [ ] `type="number"` is avoided for non-countable identifiers (ZIP/postal codes, phone numbers, credit cards, OTPs, national IDs); `inputmode="numeric"` is used instead.
- [ ] Inline validation implements *"Reward Early, Punish Late"* and links hints/errors via `aria-describedby` and `aria-invalid`.
- [ ] Password inputs include an accessible `"Show / Hide password"` toggle button (`type="button"`, `aria-pressed`) and allow pasting from password managers.
- [ ] Browser autofill (`:-webkit-autofill`) is styled cleanly so yellow/blue browser backgrounds do not clash with Dark Mode inputs.

---

## 6. The 8 Mandatory UI States Verification Matrix

Every route, view, and data component must pass inspection across all 8 operational states:

| # | UI State | Required Engineering & Design Implementation |
| :--- | :--- | :--- |
| **1** | **Loading (Skeleton)** | Geometry-matched structural skeletons reserving exact height/width (`CLS = 0`); `aria-busy="true"` on container. |
| **2** | **Empty (Zero Results / First Use)** | Clear heading explaining why it's empty, helpful context, and a 1-click primary CTA (e.g., `"Clear active filters"` or `"Create first record"`). |
| **3** | **Partial / Sparse Data** | Layout remains balanced when optional fields (avatars, secondary tags, middle names, map coordinates) are missing or when only 1 item is returned. |
| **4** | **Ideal (Standard Data)** | Clean visual hierarchy, scannable typography, `tabular-nums`, and fast interactive filtering. |
| **5** | **Extreme Overflow / Long Text** | Multi-line clamping (`line-clamp-2`), `min-w-0` + `truncate` inside Flex/Grid children, unbroken URLs (`break-words`), and large numbers (`99,999,999+`) handled without breaking layout. |
| **6** | **Error (Transient / Validation)** | Preserves user's in-progress input; displays plain-language explanation, correlation ID (if 500), and an interactive `Try Again` button. |
| **7** | **Offline / Degraded Network** | Non-intrusive banner indicating offline mode, showing cached data timestamp, and preventing silent data loss on mutations. |
| **8** | **Permission Denied / Restricted** | Friendly, explicit state when RBAC role or browser hardware permission (Geolocation, Camera, Notifications) is denied, with instructions on how to fallback (e.g., manual postal code input when GPS is denied). |
