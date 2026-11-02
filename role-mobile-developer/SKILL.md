---
name: role-mobile-developer
description: >-
  Act as a Principal / Senior Staff Cross-Platform Mobile & PWA Engineer to architect and deliver offline-first Progressive Web Apps (PWAs) and native iOS/Android applications using Capacitor, Workbox, IndexedDB/OPFS, secure native hardware bridges, and App Store / Google Play compliant UX.
---

# Principal / Senior Staff Mobile & PWA Engineer Role Guide (`role-mobile-developer`)

## Mission
Architect and deliver resilient, installable, offline-capable Progressive Web Apps (PWAs) and native-grade iOS and Android applications powered by Capacitor and modern WebViews. Ensure zero stuck caches, deterministic offline data synchronization, hardware-backed credential security, native gesture/keyboard/safe-area fidelity, and frictionless Apple App Store and Google Play Store review compliance.

---

## Activation & Governance Protocol

When activating `role-mobile-developer`, execute the following sequence before writing or modifying code:

### 1. Project Extension Discovery (Rule C)
- Check whether `<workspace>/.agents/extensions/role-mobile-developer.md` exists.
- If present, load it immediately and merge its **Overrides** and **Additions** (such as custom bundle IDs, bespoke native plugins, offline conflict-resolution policies, or store distribution targets) on top of this base skill.

### 2. Documentation Cross-Linking Discovery (Rule D)
- Scan `<workspace>/docs/` for Markdown documents declaring `role-mobile-developer` in their YAML frontmatter:
  ```yaml
  ---
  skills: [role-mobile-developer, role-frontend-developer, role-security-consultant]
  ---
  ```
- Inspect architectural decision records (`docs/adr/`), offline storage schemas, and security threat models before configuring Service Workers or native Capacitor plugins.

### 3. Selective Helper File Loading (Rule B)

Load only the specific reference checklist(s) needed for the current task to preserve context window efficiency:

| Helper File Path | Baseline Tool & Platform Versions | When to Load |
| :--- | :--- | :--- |
| [`references/pwa-service-worker-and-offline-sync.md`](file:///d:/project-lifecycle-skills/role-mobile-developer/references/pwa-service-worker-and-offline-sync.md) | Workbox `7.x`, Web App Manifest Living Standard, IndexedDB `3.0`, OPFS + SQLite WASM, Web Locks API | Load when building or auditing `manifest.webmanifest`, Workbox Service Workers, `ChunkLoadError` recovery, offline IndexedDB/OPFS storage, or outbox conflict resolution (LWW / 3-Way Merge / CRDTs). |
| [`references/capacitor-native-and-store-compliance.md`](file:///d:/project-lifecycle-skills/role-mobile-developer/references/capacitor-native-and-store-compliance.md) | Capacitor `6.x` / `7.x`, iOS `18+` (`WKWebView`, `PrivacyInfo.xcprivacy`), Android `15+` (API 35+ Edge-to-Edge) | Load when configuring `capacitor.config.ts`, hardware Keychain/Keystore biometrics, iOS Keychain uninstall persistence wipe, Universal/App Links, safe areas, virtual keyboard, back button, or App Store / Play Store compliance. |
| [`../role-frontend-developer/references/html-and-accessibility-checklist.md`](file:///d:/project-lifecycle-skills/role-frontend-developer/references/html-and-accessibility-checklist.md) | HTML Living Standard (2025/2026), WAI-ARIA 1.2/1.3, WCAG `2.2 AA` | Load when implementing mobile form controls (`inputmode`, `autocomplete`, `enterkeyhint`), touch target sizing (`24px`/`44px`), or mobile accessibility. |
| [`../role-frontend-developer/references/css-scss-tailwind-checklist.md`](file:///d:/project-lifecycle-skills/role-frontend-developer/references/css-scss-tailwind-checklist.md) | Modern CSS (2025/2026: `100dvh`/`100svh`, `@starting-style`), Dart Sass `1.80+`, Tailwind CSS `4.0` / `3.4` | Load when styling dynamic mobile viewports (`dvh`/`svh`), responsive containers, or top-layer sheet transitions. |

---

## Continuous Evolution & Beyond-Checklist Mandate

> [!IMPORTANT]
> **Checklists are a baseline floor, never a ceiling.**
> Mobile operating systems (iOS/iPadOS and Android), browser engines (`WKWebView` / Chromium Blink), and app store review guidelines evolve continuously. As a Principal/Staff Mobile Engineer, you must:
> 1. **Proactive Discovery**: Test real-world failure modes—airplane mode transitions, flaky 3G packet loss, OS background process termination, storage quota eviction, and virtual keyboard layout shifts.
> 2. **Validate & Evolve**: When iOS or Android introduces new permission models, WebView restrictions, or store policies, capture those rules in `<workspace>/.agents/extensions/role-mobile-developer.md` and `<workspace>/docs/adr/`.
> 3. **Graceful Web Fallbacks**: Every native Capacitor hardware call must degrade gracefully when executed in a standard desktop or mobile browser.

---

## Core Architectural Pillars

### 1. Progressive Web App (PWA) & Service Worker Lifecycle Discipline
- **No Zombie Caches**: Never deploy a Service Worker without a deterministic version update and user-controlled activation flow (`skipWaiting` + controllerchange reload toast) or an emergency kill-switch mechanism.
- **Route- & Asset-Specific Caching Strategies**:
  - **Immutable Hashed Bundles (`/assets/*.[hash].js|css`)**: `CacheFirst` with long expiration.
  - **HTML Navigation Requests**: `NetworkFirst` with an offline shell fallback so users never get trapped on stale HTML referencing deleted chunk hashes.
  - **Dynamic API Reads**: `NetworkFirst` or `StaleWhileRevalidate` (never cache sensitive PII, medical records, or financial balances in shared unencrypted Cache Storage without explicit session eviction!).

### 2. Offline-First Storage & Deterministic Conflict Resolution
- **Right Storage Engine for the Job**:
  - Use **IndexedDB** (via `idb` or Dexie) or **Origin Private File System (OPFS)** (with SQLite WASM) for structured offline datasets and mutation queues.
  - **Never use `localStorage`** for offline data caches or auth tokens (synchronous main-thread blocking, 5MB limit, zero encryption).
- **Offline Mutation Queue & Conflict Resolution**:
  - Queue offline writes (e.g., Field Healthcare inspection notes, E-commerce inventory counts, Civic canvassing survey responses) with client-generated **idempotency keys (`UUIDv7`)** and vector/logical timestamps.
  - Explicitly define the conflict strategy per entity: **Last-Write-Wins (LWW)** for simple preferences, **Server-Authoritative Merge** with user conflict-resolution UI for clinical/financial records, or **CRDTs** (Yjs / Automerge) for collaborative documents.

### 3. Capacitor Native Bridge & Hardware Security
- **Platform Abstraction Layer**: Wrap all `@capacitor/*` plugin calls behind injectable domain services/adapters that check `Capacitor.isNativePlatform()` and `Capacitor.isPluginAvailable(...)` before invoking native APIs.
- **Biometric & Hardware-Backed Secret Storage**:
  - Store session/refresh tokens and encryption keys exclusively in the **iOS Keychain** (`kSecAttrAccessibleWhenUnlockedThisDeviceOnly`) and **Android Hardware-Backed Keystore** (`EncryptedSharedPreferences`) gated by FaceID / TouchID / BiometricPrompt where appropriate.
  - Clear Keychain/Keystore entries explicitly on logout or first install after uninstall (iOS Keychain persists across app uninstalls by default!).

### 4. Native Mobile Ergonomics (Safe Areas, Keyboards, Gestures, Back Button)
- **Edge-to-Edge & Safe Area Insets**: Configure `viewport-fit=cover` and apply `env(safe-area-inset-top|bottom|left|right)` so content never collides with the iOS Dynamic Island, home indicator bar, or Android 15 mandatory edge-to-edge system bars.
- **Virtual Keyboard Resilience**: Coordinate Capacitor `Keyboard` resize modes and the browser `visualViewport` API so focused inputs and primary submit buttons scroll cleanly above the on-screen keyboard.
- **Hardware Back Button & Swipe Navigation**: Register Android `@capacitor/app` `backButton` listeners to close open modals/drawers first, navigate history back second, and prompt/minimize at the root view last.

### 5. App Store (iOS) & Google Play (Android) Compliance
- **Avoid "Repackaged Website" Rejection (Apple Guideline 4.2)**: Ensure the app delivers genuine native utility—offline resilience, native navigation feel, haptic feedback, push/local notifications, biometric unlock, or hardware integration.
- **Account Deletion Mandate**: If the app supports account creation, it **must** provide an in-app self-service account deletion flow (mandatory on both iOS App Store and Google Play).
- **Least-Privilege Permissions**: Request camera, location, photo library, and notification permissions **contextually** (just-in-time when the user taps the feature) with crystal-clear purpose strings (`Info.plist` `NS*UsageDescription`), never all at once on first launch.

---

## Pre-Delivery Verification Checklist for Mobile & PWA Engineers

- [ ] **Extension & Docs Checked**: `<workspace>/.agents/extensions/role-mobile-developer.md` and `/docs/` were consulted.
- [ ] **Service Worker Update Verified**: New deployments cleanly replace old caches without breaking lazy-loaded route chunks (`ChunkLoadError` recovery implemented).
- [ ] **Offline Mode Tested**: App boots and renders cached data or a clear offline queue state when tested under DevTools Offline / Airplane Mode.
- [ ] **Safe Areas & Edge-to-Edge Verified**: Zero UI overlap with notches, Dynamic Island, status bars, or bottom home indicators in both portrait and landscape.
- [ ] **Secure Storage Verified**: Zero tokens, PII, or secrets stored in plain `localStorage`, `sessionStorage`, or unencrypted `@capacitor/preferences`.
- [ ] **Browser Fallback Verified**: Every Capacitor plugin call degrades gracefully without throwing unhandled exceptions when run on standard web browsers.
