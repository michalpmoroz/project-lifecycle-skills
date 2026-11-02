# Capacitor Native Engineering, Security & App Store Compliance Reference (`references/capacitor-native-and-store-compliance.md`)

> **Baseline Versions**: Capacitor `6.x` / `7.x`, iOS `18+` (`WKWebView`, `PrivacyInfo.xcprivacy`), Android `15+` (API Level 35+ Edge-to-Edge System WebView, 16KB page size NDK), Apple App Store Review Guidelines, Google Play Developer Policy.
> **Mandate**: Load and enforce this reference whenever configuring Capacitor native bridges, biometric keychain/keystore storage, Universal/App Links, virtual keyboards, safe-area insets, hardware back buttons, or preparing iOS App Store / Google Play submissions.

> [!NOTE]
> **Continuous Evolution Mandate**: This reference is a baseline quality floor, never a ceiling. Continuously verify native hardware fallbacks in standard web browsers and audit against the latest iOS/Android OS SDK and store review changes.

---

## 1. Capacitor Native Bridge Security (`capacitor.config.ts`)

```ts
// capacitor.config.ts (Baseline: Capacitor 6.x / 7.x)
import type { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.organization.appname',
  appName: 'AppName',
  webDir: 'dist',
  server: {
    // Use https scheme on Android to ensure Web Crypto API, Secure Cookies, and Service Workers function properly
    androidScheme: 'https',
    iosScheme: 'capacitor',
    // Strictly allowlist only domains that are permitted to navigate inside the WebView
    allowNavigation: [],
  },
  plugins: {
    SplashScreen: {
      launchAutoHide: false, // Hide programmatically ONLY after initial route & auth state resolve!
    },
    Keyboard: {
      resize: 'body',
      resizeOnFullScreen: true,
    },
  },
};

export default config;
```

- **Production `server.url` Prohibition**: Never ship an App Store or Play Store build with `server.url` pointing to a remote hosted website (`https://...`). Remote `server.url` is exclusively for local live-reload development. Shipping it to production exposes the native bridge to remote XSS/MITM hijacking and triggers Apple Guideline 4.2 rejection.
- **Guard Native Bridge Calls**: Always verify `Capacitor.isNativePlatform()` before calling native-only APIs, and never expose unvalidated Capacitor plugin execution to untrusted `postMessage` frames.

---

## 2. Biometric Authentication & Hardware-Backed Secret Storage

> [!CAUTION]
> **`localStorage`, `IndexedDB`, and `@capacitor/preferences` (`NSUserDefaults` / `SharedPreferences`) store data in unencrypted plaintext XML/plist/SQLite files on disk.** Never store OAuth refresh tokens, private keys, or health/financial PII in `@capacitor/preferences`.

### 2.1 Hardware-Backed Secure Storage Rules
- Store session tokens, refresh tokens, and SQLCipher database encryption keys inside:
  - **iOS**: Keychain Services with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly` (ensures tokens are not exported to unencrypted iCloud/Finder backups or migrated to a different device).
  - **Android**: Android Keystore (`AES-256-GCM` MasterKey backing `EncryptedSharedPreferences`).
- **iOS Keychain Uninstall Persistence Pitfall**:
  - On iOS, Keychain items **survive app uninstallation**! If a user uninstalls and reinstalls the app (or hands the device over), old Keychain tokens will still exist.
  - **Mandatory Mitigation**: Store a non-sensitive boolean flag `hasInitializedInstall = true` in `@capacitor/preferences` (which *is* wiped on uninstall). On app startup, if `hasInitializedInstall` is missing, immediately **wipe all Keychain entries** before setting `hasInitializedInstall = true`.

### 2.2 Biometric Unlock Flow (FaceID / TouchID / BiometricPrompt)
- Do not merely evaluate a client-side boolean (`if (biometricSuccess) navigate('/home')`)—an attacker with Frida can hook a boolean return value.
- Instead, store the refresh token or local vault decryption key in a **biometric-bound Keychain/Keystore entry** that requires hardware cryptographic unlocking by the Secure Enclave / StrongBox TEE before the OS releases the secret bytes.

---

## 3. Deep Linking: Universal Links (iOS) & App Links (Android)

Avoid insecure custom URL schemes (`myapp://`) for authentication callbacks or sensitive links because any malicious app can register the same custom scheme and intercept tokens. Always use verified HTTPS **Universal Links (iOS)** and **Android App Links**.

### 3.1 Server Verification Files
1. **iOS (`https://<domain>/.well-known/apple-app-site-association`)**:
   - Must be served over HTTPS with `Content-Type: application/json`, **zero HTTP redirects**, and no `.json` file extension.
2. **Android (`https://<domain>/.well-known/assetlinks.json`)**:
   - Must include the production Play App Signing SHA-256 certificate fingerprint (`sha256_cert_fingerprints`).

### 3.2 Safe Deep-Link Routing Listener
```ts
import { App, type URLOpenListenerEvent } from '@capacitor/app';

App.addListener('appUrlOpen', (event: URLOpenListenerEvent) => {
  const url = new URL(event.url);
  // 1. Strictly validate hostname against allowlisted domain
  if (url.hostname !== 'app.example.org') return;

  // 2. Sanitize & validate path + search params before handing to framework router
  const targetPath = `${url.pathname}${url.search}`;
  navigateSafelyInsideZone(targetPath);
});
```

---

## 4. Virtual Keyboard, Safe Areas & Native WebView Ergonomics

### 4.1 Edge-to-Edge Safe Area Insets (iOS 18 & Android 15+)
1. Include `viewport-fit=cover` in `<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">`. (Note: **Never** set `maximum-scale=1.0` or `user-scalable=no`, which violates WCAG 2.2 AA `1.4.4 Resize Text` by breaking pinch-to-zoom!).
2. Apply CSS safe-area padding on top headers, sticky bottom tab bars, and floating action buttons:
   ```css
   .app-header {
     padding-block-start: max(0.75rem, env(safe-area-inset-top));
     padding-inline-start: max(1rem, env(safe-area-inset-left));
     padding-inline-end: max(1rem, env(safe-area-inset-right));
   }
   .bottom-tab-bar {
     padding-block-end: max(0.5rem, env(safe-area-inset-bottom));
   }
   ```
3. **Android 15 (API 35) Mandatory Edge-to-Edge**: Android 15 forces apps edge-to-edge behind system status and navigation bars. Ensure `@capacitor/status-bar` and system inset CSS variables properly pad top and bottom interactive bars.

### 4.2 Preventing iOS Input Auto-Zoom & Keyboard Overlap
- **16px Minimum Input Font Size**: iOS `WKWebView` / Safari automatically zooms the entire viewport if a user taps an `<input>`, `<select>`, or `<textarea>` with a computed `font-size` smaller than `16px` (`1rem`). Always enforce `font-size: max(16px, 1rem)` on form controls on mobile viewports.
- **Visual Viewport Synchronization**: Listen to `window.visualViewport?.addEventListener('resize', ...)` or Capacitor `Keyboard.addListener('keyboardWillShow', ...)` so sticky bottom submit buttons and chat inputs stay visible above the virtual keyboard.

### 4.3 Native Feel Polish
- **Disable Chrome Text Selection & Callouts**: Apply `user-select: none; -webkit-touch-callout: none; -webkit-tap-highlight-color: transparent;` on navigation bars, tab bars, buttons, and icons (while preserving `user-select: text` on article bodies, transaction IDs, addresses, and error codes so users can copy them).
- **Overscroll Containment**: Set `overscroll-behavior-y: none` on the root shell and `overscroll-behavior: contain` on scrollable modal/drawer bodies.
- **Tactile Haptics (`@capacitor/haptics`)**: Trigger subtle `Haptics.impact({ style: ImpactStyle.Light })` on toggle switches/tab changes and `Haptics.notification({ type: NotificationType.Success | Error })` on critical form submissions (e.g., FinTech payment sent, Healthcare vital logged).

---

## 5. Hardware Back Button & App Lifecycle Hygiene

### 5.1 Deterministic Android Back Button Priority Stack
Register a centralized back-button orchestrator via `@capacitor/app`:
1. If an overlay (Modal `<dialog>`, Action Sheet, Lightbox, or Popover) is open -> **dismiss top-most overlay** and stop propagation.
2. If a navigation Drawer / Side Sheet is open -> **close drawer**.
3. If an unsaved multi-step form is dirty -> **prompt confirmation dialog**.
4. If the router has history (`canGoBack`) -> **navigate back (`router.back()`)**.
5. If on a root tab other than Home -> **switch to Home tab**.
6. If on the Home root screen -> call `App.minimizeApp()` (or `App.exitApp()`).

### 5.2 App Switcher Privacy Snapshot Protection
For apps handling sensitive data (FinTech banking, Healthcare EHR, Legal/Civic portals):
- When `App.addListener('appStateChange', ({ isActive }) => ...)` fires with `isActive: false` (or on iOS `willResignActive`), immediately overlay a privacy blur/logo shield over the DOM so the OS task-switcher screenshot does not capture sensitive account balances or medical records on disk!

---

## 6. App Store (iOS) & Google Play Store Rejection Pitfalls Checklist

Before submitting any release candidate to App Store Connect or Google Play Console, verify every item below:

- [ ] **No Splash Screen Flash of White/Unstyled Content**: Keep `@capacitor/splash-screen` visible (`launchAutoHide: false`) until the initial framework route and authentication state have rendered, then call `SplashScreen.hide()` inside `afterNextRender` / `onMounted`.
- [ ] **Apple Guideline 4.2 (Minimum Functionality)**: App includes offline capability, native navigation ergonomics (bottom tabs, swipe back, haptics), and native platform integrations—never a bare iframe/wrapper around a marketing website.
- [ ] **Mandatory In-App Account Deletion (Apple 5.1.1(v) & Google Play Policy)**: If users can create an account in the app, a discoverable `Settings -> Delete Account` flow exists inside the app that permanently schedules/executes account and personal data deletion (not merely logging out or emailing support).
- [ ] **Sign in with Apple (Apple Guideline 4.8)**: If the app offers any third-party social login (Google, Facebook, GitHub), it **must** also offer "Sign in with Apple" with equal visual prominence.
- [ ] **Apple Privacy Manifest (`PrivacyInfo.xcprivacy`)**: All Required Reason APIs used by Capacitor and installed plugins (e.g., `NSUserDefaults` `CA92.1`, file timestamps, system boot time) are declared in `ios/App/App/PrivacyInfo.xcprivacy`.
- [ ] **Specific Permission Purpose Strings (`Info.plist`)**: Every `NS*UsageDescription` (`NSCameraUsageDescription`, `NSLocationWhenInUseUsageDescription`, `NSPhotoLibraryUsageDescription`, `NSFaceIDUsageDescription`) explains clearly and specifically *why* the user's data is needed, with concrete examples (generic strings like `"App needs camera access"` are rejected).
- [ ] **Android 15+ Target SDK & 16KB Page Size Alignment**: `targetSdkVersion >= 35`, explicit `foregroundServiceType` declarations in `AndroidManifest.xml`, and native `.so` libraries compiled with 16KB ELF alignment for Android 15+ devices.
- [ ] **Zero Broken Links & Complete Legal Metadata**: Privacy Policy URL, Terms of Service URL, and Support URL are reachable offline/online without login walls.
- [ ] **Reviewer Demo Credentials Provided**: If any part of the app requires login, active non-2FA demo credentials (or a dedicated reviewer bypass environment) are documented for App Review.
