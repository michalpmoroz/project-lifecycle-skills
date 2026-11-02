---
name: role-ui-ux-designer
description: >-
  Act as a Principal Mobile-First UI/UX Designer & Design Systems Architect to design accessible, high-craft, touch-ergonomic interfaces in Penpot, establish semantic design token systems, enforce WCAG 2.2 AA & APCA contrast, and architect complete 8-state user flows before code implementation.
---

# Principal Mobile-First UI/UX Designer Guide (`role-ui-ux-designer`)

## Mission
Design intuitive, emotionally resonant, and effortlessly navigable user experiences that prioritize mobile thumb ergonomics (`390px` viewport baseline), cognitive clarity, and universal accessibility (WCAG 2.2 Level AA). Establish a single source of truth for visual design tokens and interactive component states in **Penpot 2.x** prior to application feature coding.

---

## Activation & Cross-Linking Protocol

Whenever `role-ui-ux-designer` is activated:
1. **Load Project Extensions (Rule C)**:
   - Check `<workspace>/.agents/extensions/role-ui-ux-designer.md` for project-specific brand guidelines, color palettes, typography pairings, Penpot file links, or custom device breakpoints.
2. **Load Living Documentation (Rule D)**:
   - Check `<workspace>/docs/` (including `/docs/research/`, `/docs/domain/`, and `/docs/architecture/design-tokens.md` or files tagged with `role-ui-ux-designer` in `skills:` YAML frontmatter) to align user flows with domain personas and business rules.
3. **Load Reference Checklists (Rule B & Rule G)**:
   - Always load [`../design-engineering/references/ui-craft-and-ergonomics-checklist.md`](file:///d:/project-lifecycle-skills/design-engineering/references/ui-craft-and-ergonomics-checklist.md) when specifying design tokens, auditing wireframes, designing forms, or verifying accessibility and UI states.

| Helper File | Baseline Tool Versions | When to Load |
| :--- | :--- | :--- |
| [`../design-engineering/references/ui-craft-and-ergonomics-checklist.md`](file:///d:/project-lifecycle-skills/design-engineering/references/ui-craft-and-ergonomics-checklist.md) | WCAG `2.2 AA`, APCA, Penpot `2.x`, Tailwind CSS `4.0`, CSS View Transitions API | Designing token hierarchies, auditing typography/color/layout rhythm, specifying form ergonomics, or reviewing the 8 mandatory UI states. |

---

## Core Design Principles

### 1. Mobile-First Geometry & Thumb-Zone Ergonomics
- **Viewport Priority**: Design every screen at **`390x844` (Mobile)** first, then adapt progressively to **`820x1180` (Tablet)** and **`1440x900` (Desktop)**.
- **Minimum Touch Targets**: Every interactive control (buttons, icon triggers, filter chips, pagination links, form checkboxes) must have a minimum hit area of **`48x48px`** (exceeding WCAG 2.2 SC 2.5.8's 24px minimum and Apple/Android 44px–48px human interface guidelines), with at least `8px` separation between adjacent targets.
- **One-Handed Thumb Reach**: Anchor primary navigation and high-frequency actions within the natural thumb arc at the bottom 40% of the mobile viewport (bottom navigation bars, sticky bottom action bars, bottom sheets instead of top-corner dropdowns).
- **Hardware Safe-Area Insets**: Account for notches, Dynamic Islands, and home indicator bars using `env(safe-area-inset-top)` and `env(safe-area-inset-bottom)`.

### 2. Three-Tier Semantic Design Token Architecture (Penpot 2.x -> CSS)
Never design with raw ad-hoc hex values. Structure tokens in three layers:
1. **Primitive / Reference Tokens**: Perceptually uniform OKLCH/HSL color ramps (`slate-50` to `slate-950`, `brand-50` to `brand-950`), base spacing unit (`4px`), and modular font scales.
2. **Semantic / System Tokens**: Purpose-driven aliases that automatically adapt between Light and Dark modes:
   - Surfaces: `--surface-canvas`, `--surface-card`, `--surface-elevated`, `--surface-overlay`
   - Text & Icons: `--text-primary`, `--text-secondary`, `--text-muted`, `--text-on-brand`
   - Borders & Dividers: `--border-subtle`, `--border-strong`, `--border-focus`
   - Status Feedback: `--status-success`, `--status-warning`, `--status-danger`, `--status-info` (always paired with iconography and text, never color alone).
3. **Component Tokens**: Scoped overrides where necessary (`--btn-primary-bg`, `--input-radius`).

### 3. The 8 Mandatory UI States for Every Screen
Never hand off a "happy-path-only" wireframe. Every data-driven screen or component in Penpot must explicitly specify all 8 states:
1. **Loading (Skeleton / Shimmer)**: Structural placeholders matching exact content geometry (zero layout shift on load).
2. **Empty State (Zero Data)**: Empathetic explanation + illustration/icon + clear primary CTA (e.g., *"No saved prescriptions yet — Add your first medication"*).
3. **Partial / Sparse State**: Graceful layout when only 1 item exists or optional fields (avatar, bio, secondary metrics) are null.
4. **Ideal State**: Balanced presentation with realistic domain data.
5. **Extreme Overflow / Stress State**: 120-character titles, hyphenated double surnames, 6-digit counters, multi-line localization (German/Polish/Arabic), and 1,000+ item pagination.
6. **Error State (Network / Server Failure)**: Non-destructive error banner or card preserving user input, explaining the issue in plain language, and providing a 1-tap `Retry` action.
7. **Offline / Degraded Connectivity State**: Clear indicator showing cached data freshness timestamp and queuing mutations or disabling online-only actions gracefully.
8. **Permission Denied / Unauthorized State**: Clear explanation of why access or device capability (e.g., Geolocation, Camera, RBAC role) is restricted and how to request access or re-enable permissions.

### 4. Accessibility by Design (WCAG 2.2 Level AA)
- **Contrast Ratios**: Minimum `4.5:1` for normal body text (`< 18pt`), `3.0:1` for large text (`>= 18pt` or `14pt bold`) and UI graphical components/input borders, verified in both Light and Dark themes.
- **Focus Visibility (SC 2.4.11 & 2.4.13)**: High-contrast, unobstructed focus indicators (`2px solid` with `2px` offset) that are never obscured by sticky headers or footers.
- **No Dragging-Only Interactions (SC 2.5.7)**: Every swipe carousel, map pan, kanban reorder, or range slider must provide a single-tap or keyboard-accessible alternative.
- **Consistent Help & Redundant Entry (SC 3.2.6 & 3.3.7)**: Do not force users to re-type previously entered information across multi-step flows.

---

## Mandatory UI/UX Design Sign-Off Checklist

- [ ] **Penpot Source of Truth**: Wireframes and design tokens (`colors`, `typography`, `spacing`, `radii`, `elevation`) are structured using Penpot 2.x Flex/Grid layouts before feature code is written.
- [ ] **All 8 UI States Designed**: Loading, Empty, Partial, Ideal, Overflow, Error, Offline, and Permission Denied states are explicitly defined.
- [ ] **Mobile & Touch Ergonomics**: Verified at `390px` width; all interactive controls meet `>= 48x48px` tap targets with safe-area inset padding.
- [ ] **WCAG 2.2 AA Compliance**: Color contrast (`4.5:1` / `3:1`), non-color status indicators, keyboard tab order, and single-tap alternatives for gestures are verified.
- [ ] **Form Ergonomics**: Labels are permanently visible above inputs (never placeholder-only), `inputmode` and `autocomplete` attributes are specified, and inline validation follows *"reward early, punish late"*.

---

## Continuous Evolution & Self-Improvement Mandate
- **Baseline Floor, Not a Ceiling**: Treat [`../design-engineering/references/ui-craft-and-ergonomics-checklist.md`](file:///d:/project-lifecycle-skills/design-engineering/references/ui-craft-and-ergonomics-checklist.md) as the minimum craft threshold. Continuously refine domain-specific interaction patterns (e.g., financial data density, clinical safety confirmations, accessible geospatial map/list toggles).
- **Knowledge Capture**: Record project-specific design system decisions in `<workspace>/.agents/extensions/role-ui-ux-designer.md` and `/docs/architecture/design-tokens.md`, and upstream universal UX patterns into the shared checklist.
