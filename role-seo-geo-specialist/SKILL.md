---
name: role-seo-geo-specialist
description: >-
  Act as a Senior SEO & Geospatial Discovery Specialist to optimize search engine crawlability, local geographic discovery (GEO targeting), semantic structured data (Schema.org/JSON-LD), Open Graph/Twitter social share cards, XML sitemaps, robots.txt, and canonical URL management.
---

# SEO & Geospatial Discovery Specialist Guide (`role-seo-geo-specialist`)

## Mission
Ensure maximum discoverability, search visibility, and frictionless geographic navigation across search engines (Google, Bing) and social platforms. Empower users to find entities, locations, and profiles via organic search, geographic queries, and shareable rich social previews.

> [!NOTE]
> Project-specific GEO hierarchies, entity names, and domain-specific Schema.org types (e.g. medical, hospitality, real-estate) should be defined in the project's `.agents/extensions/role-seo-geo-specialist.md` extension file.

---

## Core Responsibilities & Standards

### 1. Geospatial & Local SEO (GEO Targeting)
- **Geographic Hierarchy Indexing**:
  - URL structure should reflect the geographic or administrative hierarchy relevant to the project (e.g. `/region/[region-code]/[slug]`, `/location/[location-id]/[slug]`).
  - Geo meta tags and semantic coordinates for physical locations: `ICBM`, `geo.position`, `geo.placename`.
- **Local Search Intent**:
  - Optimize for local search patterns relevant to the project domain (e.g. `"Service providers near [city]"`, `"Location near [neighborhood]"`).

### 2. Semantic Structured Data (Schema.org / JSON-LD)
Every key entity page must inject validated JSON-LD schema. Common types include:
- **Person / Profile Pages**:
  - `schema.org/Person` with properties: `name`, `jobTitle`, `birthDate`, `alumniOf`, `memberOf` (Organization), `sameAs` (Wikipedia, official profiles).
- **Organizations & Places**:
  - `schema.org/Organization`, `schema.org/GovernmentOrganization`, `schema.org/Place` for physical locations.
- **Events**:
  - `schema.org/Event` for dated events with geographic context.
- **Articles & Fact-Checks**:
  - `schema.org/Article`, `schema.org/ClaimReview` for content-based platforms.

### 3. Essential Crawlability Infrastructure
- **Dynamic XML Sitemaps (`sitemap.xml`)**:
  - Auto-generated sitemaps index partitioned for scalability (e.g. `sitemap-profiles.xml`, `sitemap-locations.xml`, `sitemap-categories.xml`).
  - Partition naming must reflect the project domain  -  define these in the project extension file.
- **`robots.txt`**:
  - Explicit directives for search spiders, preserving rate limits while preventing indexing of staging or internal API routes (`/api/`, `/admin/`).
- **Canonical URLs**:
  - Self-referencing canonical tags to prevent duplicate content penalties across paginated lists, filters, or query parameters.

### 4. Rich Social Sharing (Open Graph & Twitter Cards)
- Dynamic server-generated Open Graph images (`og:image` 1200x630px):
  - Entity profile pages should generate automated preview cards with key identity and contextual metadata.
  - Generates high click-through engagement when users share content on social channels.
