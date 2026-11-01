---
name: tooling-free-and-open-source
description: >-
  Enforce the Free & Open-Source First policy. Whenever evaluating or recommending tools, libraries, infrastructure, or services, pair the industry-standard solution with its best 100% free or open-source alternative.
---

# Free & Open-Source Tooling Matrix Guide

## Mission
Ensure the project can be planned, designed, developed, tested, hosted, and monitored at zero licensing cost, leveraging world-class free and open-source software (FOSS) and generous free tiers.

## The Dual-Recommendation Policy
Whenever a tooling decision arises, always document both the **Industry Benchmark** and the **Best Free / Open-Source Alternative**:

| Category | Industry Benchmark (Commercial/Paid) | Best Free & Open-Source Alternative (Zero Cost) | Key Advantages of Free Option |
| :--- | :--- | :--- | :--- |
| **UI/UX Design** | Figma / FigJam | **Penpot** (penpot.app) | 100% open source, web-based, SVG native, Flex & Grid layout engine, self-hostable |
| **Whiteboard / Diagrams** | Miro / Whimsical | **Excalidraw** / **Draw.io** | Free, privacy-respecting, exportable to SVG/PNG, embeds into GitHub PRs |
| **Code Editor** | WebStorm / Cursor Pro | **VS Code** / **VSCodium** | Free, massive extension ecosystem, built-in terminal & Git integration |
| **Mobile / Native Bridge** | React Native / Flutter (complex setups) | **Capacitor** (capacitorjs.com) | 100% open source by Ionic, works with any web framework, uses standard Xcode/Android Studio |
| **Backend as a Service (BaaS)** | Firebase (vendor lock-in) / AWS | **Supabase** (Free Tier) or **PocketBase** | Supabase offers free Postgres + Auth + Storage; PocketBase is a zero-dependency self-hostable single Go executable |
| **Database** | Managed RDS / MongoDB Atlas | **SQLite + LibSQL / Turso** or **Local PostgreSQL** | Free local execution, zero latency, easily containerized via Docker |
| **Hosting & Edge Deploy** | Vercel Pro ($20/user/mo) | **Cloudflare Pages** or **Vercel Hobby** | Cloudflare Pages provides unlimited bandwidth, global edge CDN, and 100k daily worker requests free |
| **API Client & Testing** | Postman (cloud account lock-in) | **Bruno** (usebruno.com) or **Hoppscotch** | Bruno is 100% open source, stores API collections directly as plain text in your Git repo |
| **Crash & Error Tracking** | Datadog / New Relic | **Sentry (Free Developer Tier)** or **GlitchTip** | 5,000 free events/month, rich stack traces, sourcemaps, breadcrumbs |
| **Product Analytics** | Mixpanel / Amplitude | **PostHog (Free Tier)** or **Umami** | PostHog gives 1,000,000 free events/mo + session replays; Umami is 100% open-source cookieless |
| **Uptime Monitoring** | Pingdom / Datadog Synthetics | **Better Stack (Free Tier)** or **Uptime Kuma** | Uptime Kuma is 100% open source, self-hostable with push notifications (Discord, Telegram, Email) |
| **CI/CD Automation** | CircleCI / Buildkite | **GitHub Actions** | 2,000 free minutes/month for private repos, unlimited for public repos |

