---
name: tooling-free-and-open-source
description: >-
  Enforce the Free & Open-Source First (FOSS) policy across all architecture, design, testing, hosting, database, observability, and DevOps decisions. Pair commercial industry benchmarks with the best 100% free or open-source self-hostable alternatives and verify SPDX license safety.
---

# Free & Open-Source First Tooling Matrix (`tooling-free-and-open-source`)

## Mission
Guarantee that every project built with these skills can be designed, architected, developed, tested, deployed, monitored, and scaled at **zero mandatory licensing cost ($0/month baseline)** without vendor lock-in. Whenever evaluating or recommending any tool, library, SaaS service, or infrastructure component, always apply the **Dual-Recommendation Policy**: present the commercial industry benchmark alongside the best **100% Free & Open-Source (FOSS)** or generous perpetual free-tier alternative, defaulting to the open-source solution unless the user explicitly requests otherwise.

---

## Activation & Cross-Linking Protocol

Whenever `tooling-free-and-open-source` is activated:
1. **Load Project Extensions (Rule C)**:
   - Check `<workspace>/.agents/extensions/tooling-free-and-open-source.md` for project-specific infrastructure choices, existing paid enterprise agreements, or self-hosting constraints (e.g., Docker Compose on VPS vs. Cloudflare/Vercel managed edge).
2. **Load Related Skills (Rule D)**:
   - Coordinate with `role-software-architect`, `role-tech-researcher`, `role-legal-counsel` (SPDX license compliance), and `role-devops-engineer`, and record final stack selections in `<workspace>/docs/adr/`.

> **Baseline Tooling & License Standards (Rule G)**
> - **Design & Prototyping**: Penpot `2.x` (`MPL-2.0`), Excalidraw (`MIT`), Mermaid.js `11.x` (`MIT`)
> - **Databases & BaaS**: PostgreSQL `17` (`PostgreSQL License`) + PostGIS `3.5+`, SQLite `3.47+` / LibSQL, Supabase (`Apache-2.0`), PocketBase `0.23+` (`MIT`)
> - **Testing & Quality**: Playwright `1.49+` (`Apache-2.0`), Vitest `2.x` (`MIT`), k6 `0.54+` (`AGPL-3.0` CLI runner), Bruno `1.34+` (`MIT`)
> - **License Governance**: Prefer permissive OSS licenses (`MIT`, `Apache-2.0`, `BSD`, `MPL-2.0`) for application dependencies; audit copyleft risks via `role-legal-counsel`.

---

## Comprehensive Dual-Recommendation Matrix

| Lifecycle Category | Commercial / Proprietary Benchmark | Best 100% Free & Open-Source (FOSS) / Zero-Cost Default | Architectural & Economic Advantages of the FOSS Default |
| :--- | :--- | :--- | :--- |
| **UI/UX Design & Tokens** | Figma Professional ($15–$75/editor/mo) | **Penpot 2.x** (`penpot.app` / Docker self-hosted) | 100% open-source (`MPL-2.0`), native CSS Grid & Flexbox layout engine, native W3C design tokens, SVG-first, zero seat limits. |
| **Architecture & Diagrams** | Miro / Lucidchart / Whimsical | **Mermaid.js** (in Git `/docs/`) + **Excalidraw** / **Draw.io** | Diagrams-as-code versioned directly in Markdown PRs (`Mermaid`); zero external account lock-in. |
| **Relational Database** | AWS Aurora / CockroachDB Dedicated | **PostgreSQL 17** (+ **PostGIS 3.5** / **pgvector**) or **SQLite / LibSQL (Turso)** | Industry gold standard relational + spatial + vector engine; runs identically in local Docker, VPS, or free managed tiers. |
| **Backend-as-a-Service / Auth** | Firebase / Auth0 / Clerk Pro | **Supabase** (FOSS / Free Tier), **PocketBase**, or **Better Auth / Lucia** | Zero per-MAU extortion pricing; full database ownership; standard OAuth2/Passkeys/Session support in your own DB. |
| **Headless CMS / Admin** | Contentful / Sanity Pro ($199+/mo) | **Payload CMS 3.x** (`MIT`), **Directus**, or **Strapi** | Code-first TypeScript native (`Payload 3.x`), deploys inside your Next.js/Node app using your own Postgres DB at $0 cost. |
| **Geospatial Maps & Tiles** | Google Maps Platform / Mapbox (expensive per-load fees) | **MapLibre GL JS** (`BSD-3`) + **OpenStreetMap** / **Protomaps (`PMTiles`)** | `MapLibre GL` is a 100% open-source WebGL engine; `PMTiles` hosts planet/country vector tiles as a single static file on Cloudflare R2 for $0! |
| **Geocoding & Place Search** | Google Places API | **Nominatim (OSM)** / **Photon (Komoot)** / PostGIS Full-Text + Trigram | 100% open-source OpenStreetMap data; zero per-autocomplete keystroke billing; self-hostable or cacheable. |
| **Hosting, Edge & CDN** | AWS CloudFront / Vercel Enterprise | **Cloudflare Pages & Workers**, **Vercel Hobby**, or **Coolify / Dokploy (VPS)** | Cloudflare Pages provides unlimited static bandwidth; Coolify/Dokploy turn any €4/mo VPS into a full self-hosted PaaS with preview deploys. |
| **Object & Media Storage** | AWS S3 + CloudFront Egress | **Cloudflare R2** (10GB free, **$0 egress fees**) or **MinIO / Garage** | Eliminates surprise bandwidth egress bills when serving Open Graph images, PDFs, or map tiles. |
| **Product Analytics** | Mixpanel / Amplitude / GA4 | **Umami** (`MIT`, cookieless) or **PostHog** (FOSS / 1M free events/mo) | `Umami` is < 2KB, 100% cookieless and GDPR-compliant without requiring intrusive cookie consent banners. |
| **Error & Crash Monitoring** | Datadog / New Relic | **GlitchTip** (`MIT` Sentry-compatible) or **Sentry Developer Free Tier** | Open-source compatibility with standard `@sentry/*` SDKs, sourcemaps, and stack trace scrubbing. |
| **Uptime & Status Page** | PagerDuty / Pingdom / Statuspage | **Uptime Kuma** (`MIT`) or **Better Stack Free Tier** | Self-hostable in 1 Docker container with 90+ notification providers and public status pages. |
| **API Client & Contract Testing** | Postman Enterprise (cloud sync lock-in) | **Bruno** (`MIT`) or **Hoppscotch** (`MIT`) | `Bruno` stores API collections as plain-text `.bru` files directly inside the Git repository alongside code. |
| **E2E, Unit & Load Testing** | Cypress Cloud / BrowserStack / LoadRunner | **Playwright** + **Vitest** + **Grafana k6** + **axe-core** | 100% free parallel execution, mobile device emulation, visual regression, and WCAG 2.2 AA auditing in GitHub Actions. |
| **Transactional Email** | SendGrid Pro | **Resend (Free Tier)** / **Postal (FOSS)** + **React Email / MJML** | Clean developer experience, local SMTP testing via **Mailpit** (`MIT`) in development. |

---

## Anti-Lock-In Architectural Rules (Adapter Pattern)

Even when using a free tier of a managed service (e.g., Supabase, Cloudflare R2, Resend), always isolate vendor SDK calls behind a clean internal interface/adapter (`FileStorageProvider`, `EmailSender`, `SearchIndex`):
1. **Standard Protocols Over Proprietary APIs**:
   - Prefer standard PostgreSQL wire protocol (`pg` / Drizzle / Prisma / Kysely) over proprietary NoSQL query languages.
   - Prefer S3-compatible APIs (`@aws-sdk/client-s3`) so switching between Cloudflare R2, MinIO, Backblaze B2, and AWS S3 requires changing only `S3_ENDPOINT` in `.env`.
   - Prefer standard OpenTelemetry (`OTel`) and Sentry SDK envelopes.
2. **Local Offline Development Parity**:
   - Every external dependency must be runnable locally with zero internet or paid cloud credentials (e.g., local PostgreSQL/PostGIS + Mailpit via `docker compose up -d`).

---

## Mandatory FOSS & Tooling Audit Checklist

- [ ] **Dual-Recommendation Documented**: Every stack/library proposal in `/docs/research/` and `/docs/adr/` compares the commercial benchmark against the best 100% FOSS alternative.
- [ ] **Zero Surprise Billing Risks**: Geospatial maps (`MapLibre` + `PMTiles`/OSM), media storage (`R2`/S3), and analytics (`Umami`) are architected to prevent runaway per-request billing spikes (denial-of-wallet attacks).
- [ ] **SPDX License Verified**: Confirmed with `role-legal-counsel` that no library linked into the application introduces incompatible viral copyleft (`AGPL-3.0`/`SSPL`) unless intended.
- [ ] **1-Command Local Dev (`docker compose`)**: The entire stack can be booted locally by a developer or CI runner at $0 cost.

---

## Continuous Evolution & Self-Improvement Mandate
- **Baseline Floor, Not a Ceiling**: The FOSS ecosystem evolves rapidly. Continuously evaluate emerging open-source tools during `role-tech-researcher` spikes, record project-specific tool overrides in `<workspace>/.agents/extensions/tooling-free-and-open-source.md`, and upstream superior FOSS alternatives into this matrix.
