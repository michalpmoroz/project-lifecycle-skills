# Technical SEO, Geospatial Indexing & Schema.org Checklist

> **Baseline Tooling & Standard Versions (Rule G)**
> - **Search Engine Guidelines**: Google Search Essentials (`2026`), Bing Webmaster Guidelines
> - **Structured Data Vocabulary**: Schema.org `v28.0+` (JSON-LD `1.1` serialization)
> - **Robots Exclusion Protocol**: IETF RFC `9309` (`robots.txt`) + `X-Robots-Tag` HTTP headers
> - **Instant Indexing Protocol**: IndexNow API `1.0`
> - **Social Metadata**: Open Graph Protocol `1.0` + X/Twitter Card Markup
> - **Geospatial Standards**: IETF RFC `7946` (GeoJSON), PostGIS `3.5+` (`GEOGRAPHY(Point, 4326)`, `GIST` spatial indexes)

---

## 1. Crawl Budget, `robots.txt` & Indexing Directives

### `robots.txt` Architecture (RFC 9309)
- **Never Block JS/CSS Bundles**: Modern search crawlers (Googlebot Web Rendering Service) must fetch CSS and JS bundles (`/_next/static/`, `/assets/`) to verify mobile usability and Core Web Vitals.
- **Block Non-Public & Infinite Crawl-Trap Routes**: Disallow `/api/`, `/admin/`, `/auth/`, `/checkout/`, internal user dashboards, and raw facet permutations.
- **AI Crawler Governance**: Explicitly configure policies for search vs. training bots (`OAI-SearchBot`, `PerplexityBot`, `Google-Extended`, `CCBot`) according to the project's legal/licensing posture in `.agents/extensions/role-seo-geo-specialist.md`.

```text
# /public/robots.txt
User-agent: *
Allow: /
Disallow: /api/
Disallow: /admin/
Disallow: /auth/
Disallow: /*?sort=*
Disallow: /*& filter=*

Sitemap: https://example.com/sitemap-index.xml
```

### Meta Robots vs. `robots.txt` Pitfall
- **Critical Rule**: If a page already indexed in Google needs to be removed from search results, **do NOT block it in `robots.txt` first** — if Googlebot is blocked by `robots.txt`, it cannot crawl the page to see the `<meta name="robots" content="noindex" />` tag or `X-Robots-Tag: noindex` HTTP header!
- Always serve `noindex, follow` (or `404`/`410 Gone` for deleted pages) to de-index URLs cleanly.

### Checklist — Crawlability & Directives
- [ ] `robots.txt` is reachable at `/robots.txt` with `200 OK` (`text/plain`) and links to the absolute `Sitemap:` URL.
- [ ] Zero accidental `noindex` tags left over from staging/preview environments on production domain routes.
- [ ] Deleted entities return proper HTTP `404 Not Found` or `410 Gone` (never a "Soft 404" that returns HTTP `200 OK` with an empty state saying *"Record not found"*).

---

## 2. Partitioned XML Sitemaps & IndexNow

### Sitemap Index Partitioning
Large or multi-entity platforms must split sitemaps by entity type and cap each file well below the protocol limits (`<= 45,000` URLs and `< 50MB` uncompressed per XML file):
- `https://example.com/sitemap-index.xml`
  - `sitemap-static.xml` (Homepage, About, Methodology, Legal, Contact)
  - `sitemap-categories.xml` / `sitemap-regions.xml` (Taxonomies, Regions, Districts, Cities)
  - `sitemap-entities-1.xml` (Products / Profiles / Practitioners / Listings — paginated by 25,000)
  - `sitemap-articles.xml` (Editorial content, News, Guides, Case Studies)

### `<lastmod>` Integrity Rule
- **Never set `<lastmod>` to `new Date().toISOString()` on every request!** Search engines explicitly ignore `<lastmod>` across an entire domain if they detect that the timestamp changes when the page content has not actually changed.
- Bind `<lastmod>` strictly to the entity's actual database `updated_at` timestamp (`YYYY-MM-DDTHH:mm:ssZ`).
- Omit `<changefreq>` and `<priority>` — Googlebot officially ignores both.

### Checklist — Sitemaps
- [ ] Every URL in `sitemap-*.xml` returns HTTP `200 OK`, is indexable (`index, follow`), and matches its exact `<link rel="canonical">` URL (zero redirects or non-canonical URLs inside sitemaps).
- [ ] `<lastmod>` reflects true database modification timestamps.
- [ ] Dynamic entity creation/updates trigger cache revalidation and optional `IndexNow` pings for rapid indexing.

---

## 3. Canonicalization, Pagination & International SEO (`hreflang`)

### Canonicalization Rules
- Enforce a single canonical host (`https://` and either `www` or apex via `301 Permanent Redirect`), lowercase URL slugs (`/regions/north-west`, never `/Regions/North_West`), and consistent trailing-slash behavior.
- **Paginated Series (`?page=2` or `/page/2`)**:
  - **CRITICAL ANTI-PATTERN**: Never canonicalize Page 2, Page 3, etc. back to Page 1! Doing so tells search engines that Page 2 is a duplicate of Page 1, causing crawlers to drop all links and entities listed on Page 2+.
  - Every paginated page (`/directory?page=2`) must have a **self-referencing canonical** (`<link rel="canonical" href="https://example.com/directory?page=2" />`) and crawlable `<a href="/directory?page=3">` pagination links.

### Multilingual & Multi-Regional `hreflang`
For multilingual platforms:
- Every localized page must include bidirectional `<link rel="alternate" hreflang="<lang-region>" href="..." />` tags pointing to all language variants **including itself**, plus an `x-default` fallback URL.

```html
<link rel="canonical" href="https://example.com/en/clinics/berlin-mitte" />
<link rel="alternate" hreflang="en" href="https://example.com/en/clinics/berlin-mitte" />
<link rel="alternate" hreflang="de" href="https://example.com/de/kliniken/berlin-mitte" />
<link rel="alternate" hreflang="x-default" href="https://example.com/en/clinics/berlin-mitte" />
```

---

## 4. Schema.org v28+ JSON-LD Structured Data

Inject validated JSON-LD via `<script type="application/ld+json">`, escaping `<` as `\u003c` to prevent XSS breakout inside script tags:

```typescript
// Safe JSON-LD serialization helper
export function serializeJsonLd(schema: Record<string, unknown>): string {
  return JSON.stringify(schema).replace(/</g, '\\u003c');
}
```

### Multi-Domain Entity Schema Mapping

| Domain | Primary Schema.org Types | Required & High-Impact Properties |
| :--- | :--- | :--- |
| **Universal (All Pages)** | `WebSite`, `Organization`, `WebPage`, `BreadcrumbList` | `@id`, `name`, `url`, `logo`, `sameAs`, `itemListElement` (positioned `1..N`). |
| **E-commerce / SaaS** | `Product`, `SoftwareApplication`, `Offer`, `AggregateRating` | `name`, `image`, `description`, `sku`, `brand`, `offers` (`price`, `priceCurrency`, `availability`: `https://schema.org/InStock`). |
| **Healthcare / Clinics** | `MedicalClinic`, `Physician`, `MedicalSpecialty` | `name`, `address` (`PostalAddress`), `geo` (`GeoCoordinates`), `telephone`, `openingHoursSpecification`, `medicalSpecialty`. |
| **Civic / Public Directory** | `Person`, `GovernmentOrganization`, `AdministrativeArea`, `Event` | `name`, `jobTitle`, `affiliation` / `memberOf`, `worksFor`, `birthDate`, `homeLocation` / `spatialCoverage`, `sameAs` (Wikidata/official URLs). |
| **Editorial / Fact-Check** | `Article`, `NewsArticle`, `ClaimReview`, `FAQPage` | `headline`, `datePublished`, `dateModified`, `author` (`Person`/`Organization` with `url`), `claimReviewed`, `reviewRating`. |

### Connected `@graph` Example
```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "BreadcrumbList",
      "@id": "https://example.com/regions/bavaria/munich#breadcrumb",
      "itemListElement": [
        { "@type": "ListItem", "position": 1, "name": "Regions", "item": "https://example.com/regions" },
        { "@type": "ListItem", "position": 2, "name": "Bavaria", "item": "https://example.com/regions/bavaria" },
        { "@type": "ListItem", "position": 3, "name": "Munich", "item": "https://example.com/regions/bavaria/munich" }
      ]
    },
    {
      "@type": "AdministrativeArea",
      "@id": "https://example.com/regions/bavaria/munich#place",
      "name": "Munich District",
      "geo": {
        "@type": "GeoCoordinates",
        "latitude": 48.1351,
        "longitude": 11.5820
      },
      "containedInPlace": {
        "@type": "AdministrativeArea",
        "name": "Bavaria"
      }
    }
  ]
}
```

### Checklist — JSON-LD
- [ ] JSON-LD uses `\u003c` escaping so user/CMS strings containing `</script>` cannot execute XSS.
- [ ] Every structured data property matches visible text rendered on the page (Google spam policies forbid hidden structured data that contradicts visible page content).
- [ ] Every detail page includes a `BreadcrumbList` schema matching the visible breadcrumb navigation.

---

## 5. Geospatial SEO & Accessible Map Discovery

### Overcoming the Canvas/WebGL Indexing Blind Spot
Interactive maps (`MapLibre GL`, `Leaflet`, `Deck.gl`) render markers inside `<canvas>` or dynamic SVGs that search crawlers do not traverse. To dominate local/geographic queries (`"providers in [district]"`, `"events near [city]"`):

1. **Hierarchical Geographic Landing Pages**:
   - Create dedicated, server-rendered landing pages for every administrative level (`/country/region/district/locality`) containing:
     - Unique descriptive summary statistics (e.g., total verified entities, key metrics, regional overview).
     - Crawlable HTML links (`<a href="...">`) to child localities and all entities located within that boundary.
2. **Synchronized Map + Semantic HTML List**:
   - On map search pages, always server-render the initial list of results as semantic `<article>` cards with `<a href>` links alongside the map viewport.
3. **Spatial Database Performance (PostGIS / GeoJSON)**:
   - Store coordinates in WGS84 (`SRID 4326`) with a spatial `GIST` index (`CREATE INDEX idx_entities_location ON entities USING GIST (location);`).
   - Pre-simplify complex administrative boundary GeoJSON polygons (`ST_SimplifyPreserveTopology`) for web delivery so initial map payloads stay `< 100KB` gzipped and do not degrade Core Web Vitals (`LCP` / `INP`).

---

## 6. Dynamic Open Graph & Social Preview Cards

Every public route must include complete Open Graph and X/Twitter metadata:
- `og:title` (`<= 60` characters, high-signal entity name + context)
- `og:description` (`120–160` characters)
- `og:url` (must match the canonical URL)
- `og:type` (`website`, `article`, `profile`, or `product`)
- `og:image` (Absolute HTTPS URL to a `1200x630px` PNG/JPEG/WebP image, `< 300KB`, generated dynamically via edge/server image generation such as `next/og` / `@vercel/og` / `satori` / `sharp`)
- `og:image:width` (`1200`), `og:image:height` (`630`), `og:image:alt` (descriptive alt text for accessibility)
- `twitter:card` (`summary_large_image`)

---

## 7. Generative Engine Optimization (GEO / AI Search Readiness)

AI answer engines extract and cite sources that provide high-authority, structured, unambiguous facts:
- **Inverted Pyramid Key Facts Block**: Render an explicit "Key Facts / At a Glance" semantic definition list (`<dl><dt>...</dt><dd>...</dd></dl>`) or table near the top of entity profiles (e.g., Official Role/Specialty, Jurisdiction/Location, Verification Status, Last Audited Date, Primary Official Sources).
- **Authoritative Outbound Citations**: Link claims, statistics, legal statuses, or clinical guidelines directly to primary institutional sources using descriptive anchor text and `<cite>` tags.
- **Explicit Timestamps**: Always display both `Published` and `Last Verified / Updated` dates using `<time datetime="2026-11-02">`.
