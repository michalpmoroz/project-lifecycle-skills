# PWA Manifest, Workbox 7 Service Worker & Offline Sync Reference (`references/pwa-service-worker-and-offline-sync.md`)

> **Baseline Versions**: Workbox `7.x`, Web App Manifest Living Standard, IndexedDB `3.0`, Origin Private File System (OPFS) + SQLite WASM, Web Locks API, iOS `18+` (`WKWebView` / Safari), Android `15+` Chromium.
> **Mandate**: Load and enforce this reference whenever building, configuring, or reviewing Progressive Web App manifests, Service Workers, caching strategies, or offline storage and conflict-resolution engines.

> [!NOTE]
> **Continuous Evolution Mandate**: This reference is a baseline quality floor, never a ceiling. Proactively test offline transitions, flaky 3G packet loss, quota eviction, and multi-tab synchronization races.

---

## 1. Web App Manifest & Installability (`manifest.webmanifest`)

### 1.1 Production-Grade Manifest Configuration
- Serve `manifest.webmanifest` with `Content-Type: application/manifest+json`.
- **Never combine `"purpose": "any maskable"` on a single icon entry!** Combining them forces OS launchers to slice full-bleed icons with unsafe crop masks. Always provide dedicated `"purpose": "any"` icons and separate `"purpose": "maskable"` icons (with key artwork kept within the inner 80% safe-zone circle).

```json
{
  "id": "/",
  "name": "Acme Field Operations Portal",
  "short_name": "FieldOps",
  "description": "Offline-capable field inspection, scheduling, and reporting workspace.",
  "start_url": "/?source=pwa",
  "scope": "/",
  "display": "standalone",
  "display_override": ["window-controls-overlay", "standalone", "minimal-ui"],
  "orientation": "portrait-primary",
  "background_color": "#0f172a",
  "theme_color": "#0f172a",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png", "purpose": "any" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png", "purpose": "any" },
    { "src": "/icons/icon-maskable-512.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ]
}
```

### 1.2 iOS Safari / Home Screen Web App Tags
Include `<link rel="apple-touch-icon" sizes="180x180" href="/icons/apple-touch-icon.png">` and `<meta name="apple-mobile-web-app-capable" content="yes">` (alongside standard `<meta name="mobile-web-app-capable" content="yes">`) and `<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">`.

---

## 2. Service Worker Lifecycle, Caching & Update Flows (Workbox 7.x)

### 2.1 Preventing Stuck / Zombie Caches
1. **HTTP Headers on `sw.js`**: The Service Worker entry script (`sw.js` or `ngsw-worker.js`) and `index.html` must **never** be cached with immutable long `max-age` headers at the CDN/server layer. Enforce `Cache-Control: no-cache, no-store, must-revalidate` on `sw.js`.
2. **Controlled Activation (`Prompt-for-Update` Pattern)**:
   - Avoid calling `self.skipWaiting()` unconditionally if the app uses lazy-loaded route chunks. Unconditional `skipWaiting()` mid-session evicts previous precached JS chunks while the running tab still references the old asset manifest!
   - Instead, detect when a new Service Worker reaches the `waiting` state, display a non-intrusive toast (`"A new version is ready — Reload"`), send `{ type: 'SKIP_WAITING' }` to the waiting worker on user confirmation, and reload inside `navigator.serviceWorker.addEventListener('controllerchange', ...)`.
3. **Lazy Chunk Mismatch Recovery (`ChunkLoadError`)**:
   - Whenever a dynamic `import()` fails due to a newly deployed build invalidating old chunk hashes (e.g., Vite's `vite:preloadError` event or Webpack/Angular chunk load error), catch the error and trigger a single controlled `window.location.reload()` (guarded by a `sessionStorage` timestamp so an actual network outage does not cause an infinite reload loop).

### 2.2 Caching Strategy Matrix

| Resource Type | Workbox Strategy | Security & Expiration Rules |
| :--- | :--- | :--- |
| **Hashed Static Bundles** (`/assets/*.[hash].js\|css\|woff2`) | `Precache` or `CacheFirst` | Safe to cache for 1 year (`maxAgeSeconds: 31536000`). |
| **HTML App Shell / Navigation** | `NetworkFirst` (`networkTimeoutSeconds: 3`) | Falls back to precached `/offline.html` or app shell when offline. |
| **Public Reference Data** (E-commerce catalog, Civic polling locations, Healthcare clinic list) | `StaleWhileRevalidate` | Cap with `ExpirationPlugin({ maxEntries: 200, maxAgeSeconds: 86400 })`. |
| **Authenticated User / Sensitive Data** (FinTech balances, Patient charts, Private messages) | `NetworkOnly` (or encrypted IndexedDB with explicit logout wipe) | **NEVER cache authenticated PII/PHI/PCI responses in shared plaintext Cache Storage** unless purged deterministically on `logout` and session expiry! |

### 2.3 Background Sync & Cross-Platform Replay
- Chromium browsers support the native Background Sync API (`Workbox BackgroundSyncPlugin` / `registration.sync.register('outbox-sync')`).
- **iOS `WKWebView` / Safari Fallback Mandatory**: Because iOS does not reliably execute background sync when the PWA/app is closed, **always** supplement Background Sync by flushing the IndexedDB mutation outbox on:
  1. `window.addEventListener('online', flushOutbox)`
  2. `document.addEventListener('visibilitychange', ...)` when returning to `visible`
  3. Capacitor `App.addListener('appStateChange', ({ isActive }) => { if (isActive) flushOutbox(); })`

### 2.4 Logout Cache Purge & Emergency Service Worker Kill-Switch
- **Mandatory Logout Purge**: When a user signs out, explicitly delete all user-scoped runtime caches (`await caches.delete('api-user-cache')`) and clear user IndexedDB tables so a subsequent user on a shared tablet/kiosk cannot inspect cached records.
- **Emergency Kill-Switch Readiness**: Keep a documented static `sw.js` kill-switch snippet ready that calls `self.skipWaiting()`, clears all `caches.keys()` in `activate`, calls `await self.registration.unregister()`, and reloads controlled clients (`client.navigate(client.url)`) in case a defective Service Worker is ever deployed.

---

## 3. Offline Storage Architecture & Conflict Resolution

### 3.1 Storage Engine Selection & Eviction Defense
- **Prohibited for Offline Data**: `localStorage` and `sessionStorage` (synchronous, blocks main thread, 5MB limit, strings only).
- **Recommended Engines**:
  - **IndexedDB** (via `Dexie.js` or `idb`) for structured entities, blobs, and outbox queues.
  - **Origin Private File System (OPFS) + SQLite WASM** for relational queries over large offline datasets (50MB–500MB+).
- **Request Persistent Storage**: Call `await navigator.storage.persist()` after user onboarding/login so mobile OS storage pressure does not silently evict IndexedDB data, and monitor usage via `await navigator.storage.estimate()`.

### 3.2 Offline Mutation Queue & Conflict Resolution Patterns
Every offline-capable write operation must be wrapped in an **Outbox Command Envelope**:

```ts
export interface OfflineMutationEnvelope<TPayload = unknown> {
  readonly mutationId: string;       // Client-generated UUIDv7 (time-ordered idempotency key)
  readonly entityType: string;       // e.g., 'inspection_report' | 'cart_item' | 'ballot_check'
  readonly entityId: string;
  readonly baseVersion: number;      // Optimistic concurrency version (ETag / row_version) read when edited
  readonly createdAtIso: string;
  readonly payload: TPayload;
  readonly retryCount: number;
  readonly status: 'pending' | 'syncing' | 'conflict' | 'failed_permanent';
}
```

Choose the right conflict resolution policy for the domain:
1. **Idempotent Upsert + Last-Write-Wins (LWW)**: Suitable for user preferences, read receipts, or single-user drafts.
2. **Optimistic Concurrency (`If-Match` / `baseVersion`) + 3-Way Merge / User Resolution UI**: Mandatory for high-stakes records (e.g., Healthcare clinical notes, FinTech expense approvals, Civic precinct tally sheets). If the server returns `409 Conflict`, mark the envelope `status: 'conflict'` and present a side-by-side comparison ("Your Offline Changes" vs. "Latest Server Version") rather than silently overwriting data.
3. **CRDTs (Yjs / Automerge)**: Suitable for collaborative multi-user real-time documents and checklists.

### 3.3 Multi-Tab Outbox Coordination (`navigator.locks`)
When a PWA is open across multiple browser tabs, firing `window.addEventListener('online', flushOutbox)` simultaneously in every tab causes duplicate concurrent outbox replay. Guard outbox flushing with the **Web Locks API** (`navigator.locks`):

```ts
export async function flushOutboxExclusively(runFlush: () => Promise<void>): Promise<void> {
  if ('locks' in navigator) {
    await navigator.locks.request('offline-outbox-sync', { ifAvailable: true }, async (lock) => {
      if (!lock) return; // Another tab is already flushing the outbox
      await runFlush();
    });
  } else {
    await runFlush();
  }
}
```
