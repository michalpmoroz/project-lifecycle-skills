# Multi-Tier Caching, Network Transport, Edge & CI Performance Gates Checklist

> **Baseline Infrastructure, Caching & Transport Versions**:
> - **In-Memory & Edge Caching**: Redis 7.4+ / Valkey 8.0+, Cloudflare Workers / Cache API / Varnish 7.x
> - **Network Transport**: HTTP/2, HTTP/3 (QUIC / RFC 9114), TLS 1.3 (RFC 8446), HTTP `103 Early Hints` (RFC 8297)
> - **Containers & CI/CD Gates**: Docker 27+ (BuildKit / Distroless), Kubernetes 1.32+, Lighthouse CI (`@lhci/cli`) 0.14+

---

## 1. Four-Tier Caching Hierarchy

Design read-heavy systems so requests are served at the outermost possible tier:

| Tier | Mechanism | Target Latency | Typical TTL & Strategy |
| :--- | :--- | :--- | :--- |
| **Tier 1: Browser Cache** | `Cache-Control: public, max-age=31536000, immutable` | `0ms` (Disk/Memory) | Content-hashed static bundles (`.a1b2c3.js`, `.woff2`, `.avif`). |
| **Tier 2: CDN Edge Cache** | `Cache-Control: public, s-maxage=60, stale-while-revalidate=600` + Cache Tags | `5ms–25ms` | Public HTML pages, catalog listings, public APIs, GeoJSON maps. |
| **Tier 3: Distributed App Cache** | Redis 7.4 / Valkey 8 (Cache-Aside / Read-Through) | `0.5ms–2ms` | Authenticated session state, expensive computed DTOs, rate-limit counters. |
| **Tier 4: In-Process / Request Memo** | Node/Go LRU (`lru-cache`) or Request-Scoped `DataLoader` | `< 0.05ms` | Feature flags, public signing keys (JWKS), per-request deduplication. |

---

## 2. CDN Cache-Key Normalization (Stripping Tracking Query Parameters)

- **The Problem**: Marketing campaigns, social media shares, and ad networks append unique per-click query parameters (`?utm_source=newsletter&fbclid=IwAR123...&gclid=Cj0KC...&_ga=...`) to public URLs. By default, CDNs include the entire query string in the Cache Key—meaning every visitor with a unique `fbclid` or `gclid` bypasses the CDN cache (`CF-Cache-Status: MISS`) and hits the origin server directly, dropping edge cache hit ratios from **95%+ down to < 20%** during high-traffic campaigns.
- **Rule**: Normalize the CDN Cache Key at the Edge (via Cloudflare Cache Rules, CloudFront Cache Policies, Fastly VCL, or Edge Middleware) to **strip all marketing/analytics tracking parameters** (`utm_*`, `fbclid`, `gclid`, `msclkid`, `mc_cid`, `mc_eid`, `ref`, `_ga`, `_gl`) or allowlist only known functional query parameters (`page`, `limit`, `sort`, `category`, `q`, `lang`).

```typescript
// Example: Edge Worker Cache-Key Normalization (Stripping Tracking Params)
const TRACKING_PARAMS = new Set(['fbclid', 'gclid', 'msclkid', 'mc_cid', 'mc_eid', '_ga', '_gl']);

export function buildNormalizedCacheUrl(requestUrl: string): string {
  const url = new URL(requestUrl);
  for (const key of [...url.searchParams.keys()]) {
    if (key.startsWith('utm_') || TRACKING_PARAMS.has(key.toLowerCase())) {
      url.searchParams.delete(key);
    }
  }
  // Sort remaining functional query params alphabetically so ?a=1&b=2 and ?b=2&a=1 share one cache key
  url.searchParams.sort();
  return url.toString();
}
```

---

## 3. Cache Stampede (Thundering Herd) Prevention

- **The Problem**: When a popular cache key expires (e.g., homepage hero feed, election night live scoreboard, flash-sale product inventory) under 5,000 requests/sec, all 5,000 requests simultaneously miss the cache and hammer PostgreSQL with the identical expensive query, causing a cascading database outage.
- **Remedies (Must implement at least one on hot keys)**:
  1. **Singleflight / In-Flight Coalescing**: Deduplicate concurrent in-process requests for the same key so only **1 request** queries the database/Redis while the other 4,999 await the same Promise.
  2. **Stale-While-Revalidate (Soft TTL + Hard TTL)**: Store the cached value in Redis with a logical `softExpireAt` timestamp inside the payload and a longer physical Redis `EX` TTL. When a request sees `now > softExpireAt`, it immediately returns the slightly stale value to the user (0ms latency) and acquires a non-blocking Redis lock (`SET lock:key 1 NX EX 5`) to refresh the cache asynchronously in the background.
  3. **TTL Jitter**: Always add random jitter (`baseTtlSeconds + Math.floor(Math.random() * 60)`) when bulk-populating cache keys so thousands of keys never expire at the exact same second.

```typescript
// Singleflight (In-Flight Promise Coalescing) Pattern
const inFlightRequests = new Map<string, Promise<unknown>>();

export async function getWithSingleflight<T>(
  key: string,
  fetcher: () => Promise<T>
): Promise<T> {
  const existing = inFlightRequests.get(key) as Promise<T> | undefined;
  if (existing) {
    return existing; // Coalesce concurrent callers onto the single active Promise!
  }

  const promise = fetcher().finally(() => {
    inFlightRequests.delete(key);
  });

  inFlightRequests.set(key, promise);
  return promise;
}
```

---

## 4. Resilience & Tail-Latency Protection (Timeouts, Request Hedging, Circuit Breakers)

- **Rule 1 (Explicit Timeouts)**: Never make an outbound `fetch()`, gRPC, or SDK call without an explicit `AbortSignal.timeout(ms)`. Unbounded external calls tie up worker threads/sockets when a third-party gateway (payment processor, national ID verifier, map geocoder) degrades.
- **Rule 2 (Request Hedging for P99 Tail Latency)**: On latency-critical, **idempotent read** operations (e.g., fetching feature flags, user entitlements, pricing rules, or read-replica lookups), if the primary request has not responded within the P95 latency threshold (e.g., `50ms`), fire a **hedged backup request** in parallel and cancel the loser via `AbortController` as soon as the fastest response arrives. This cuts P99/P99.9 tail latency caused by transient network packet loss or replica GC pauses by 60%–80% while adding < 5% total cluster load.
- **Rule 3 (Circuit Breakers)**: Wrap non-critical external dependencies in a Circuit Breaker so degraded downstream services fail fast (`< 1ms`) with a graceful fallback instead of adding timeout latency to every user request.

```typescript
// Request Hedging Pattern for Idempotent Reads (Cuts P99 Tail Latency)
export async function hedgedRead<T>(
  executeRead: (signal: AbortSignal, replicaIndex: number) => Promise<T>,
  hedgeDelayMs = 45
): Promise<T> {
  const primaryController = new AbortController();
  const backupController = new AbortController();

  return new Promise<T>((resolve, reject) => {
    let settled = false;
    let failures = 0;

    const handleSuccess = (result: T, winner: 'primary' | 'backup') => {
      if (settled) return;
      settled = true;
      clearTimeout(hedgeTimer);
      if (winner === 'primary') backupController.abort();
      else primaryController.abort();
      resolve(result);
    };

    const handleFailure = (err: unknown) => {
      failures += 1;
      if (failures === 2 && !settled) {
        settled = true;
        reject(err);
      }
    };

    // 1. Launch primary read immediately
    executeRead(primaryController.signal, 0)
      .then((res) => handleSuccess(res, 'primary'))
      .catch(handleFailure);

    // 2. If primary hasn't resolved within hedgeDelayMs (P95), launch backup read on replica 1
    const hedgeTimer = setTimeout(() => {
      if (!settled) {
        executeRead(backupController.signal, 1)
          .then((res) => handleSuccess(res, 'backup'))
          .catch(handleFailure);
      }
    }, hedgeDelayMs);
  });
}
```

---

## 5. HTTP/2 Multiplexing, HTTP/3 (QUIC) & TLS 1.3 Tuning

- **Rule**: Enable **HTTP/2** and **HTTP/3 (`Alt-Svc: h3=":443"; ma=86400`)** at the ingress/CDN edge.
- **Why**: HTTP/3 runs over **QUIC (UDP)**, eliminating TCP head-of-line blocking on lossy mobile cellular networks and reducing connection establishment to **1-RTT (or 0-RTT on session resumption)** with **TLS 1.3** and **OCSP Stapling**.
- **Early Hints (`103 Early Hints`)**: Configure the CDN/reverse proxy to emit HTTP `103 Early Hints` with `Link: </assets/main.css>; rel=preload; as=style` while the backend server is still generating the dynamic HTML response, utilizing network think-time to warm the browser cache.

---

## 6. Container Image Size & Cold-Start Optimization

- **Rule**: Use multi-stage Docker builds (`FROM node:22-alpine AS builder` → `FROM gcr.io/distroless/nodejs22-debian12` or minimal `alpine` runtime stage).
- **Impact**: Stripping build toolchains, `devDependencies`, and source artifacts shrinks container images from `> 1.2 GB` to `< 120 MB`, cutting Kubernetes Horizontal Pod Autoscaler (HPA) or serverless container pull/cold-start times from 45 seconds to **< 3 seconds**.

---

## 7. Automated CI/CD Performance Budgets & Regression Gates

- **Rule**: Performance cannot rely solely on manual spot checks. Enforce automated regression gates in GitHub Actions / CI:
  1. **Lighthouse CI (`lhci autorun`)**: Assert mobile performance score `>= 0.90`, LCP `<= 2500ms`, CLS `<= 0.1`, and Total Blocking Time (TBT) `<= 200ms` on key routes before merging PRs.
  2. **Bundle Size Budget Check**: Fail CI if initial JS chunk size grows beyond the configured budget (e.g., `150 KB` compressed) or increases by `> 5%` in a single PR without architectural sign-off.
  3. **Database Query Count Assertions**: In backend integration tests, assert that list endpoints execute a constant number of SQL queries ($O(1)$, e.g., `<= 3 queries`) whether the seeded test database contains 5 records or 50 records (automatically catching any N+1 regression before it reaches production).

```json
// Example: .lighthouserc.json CI Budget Assertions
{
  "ci": {
    "collect": {
      "numberOfRuns": 3,
      "settings": { "preset": "mobile" }
    },
    "assert": {
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.9 }],
        "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }],
        "cumulative-layout-shift": ["error", { "maxNumericValue": 0.1 }],
        "total-blocking-time": ["error", { "maxNumericValue": 200 }],
        "resource-summary:script:size": ["error", { "maxNumericValue": 153600 }]
      }
    }
  }
}
```
