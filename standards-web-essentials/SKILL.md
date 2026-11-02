---
name: standards-web-essentials
description: >-
  Standard boilerplate catalogue and mandatory pre-backlog audit checklist of universal web application features (Sitemaps, Robots.txt, SEO/Open Graph, 404/500 Error Boundaries, Cookie/Privacy Notices, Terms of Service, Accessibility Statements, Contact/Dispute Forms, Security.txt, Health Checks, and PWA Manifests).
---

# Universal Web Essentials & Boilerplate Checklist (`standards-web-essentials`)

## Mission
Ensure every web application ships with the complete foundational legal, operational, navigational, discovery, accessibility, and resilience infrastructure required of a production-grade platform. Prevent the classic failure mode where a team builds complex domain features while forgetting custom `404`/`500` pages, `sitemap.xml`, `security.txt`, legal disclosures, or contact/rectification channels.

---

## Activation & Cross-Linking Protocol

Whenever `standards-web-essentials` is activated (mandated during Phase 1/2 of `init-project` and before finalizing any MVP backlog):
1. **Load Project Extensions (Rule C)**:
   - Check `<workspace>/.agents/extensions/standards-web-essentials.md` for project-specific route slugs (including localized slugs such as `/polityka-prywatnosci` or `/impressum` where configured), custom healthcheck endpoints, or regulatory additions.
2. **Load Living Documentation (Rule D)**:
   - Record the agreed Web Essentials scope matrix in `<workspace>/docs/architecture/` or `<workspace>/docs/research/` with `skills: [standards-web-essentials, role-product-owner, role-seo-geo-specialist, role-legal-counsel]` frontmatter, and create tracked GitHub Issues for every required item.

> **Baseline Web Standards & RFCs (Rule G)**
> - **Search & Indexing**: RFC `9309` (`robots.txt`), Sitemaps XML `0.9`, Schema.org `v28+`, Open Graph `1.0`
> - **Security Disclosure**: IETF RFC `9116` (`/.well-known/security.txt`)
> - **Problem Details & Errors**: IETF RFC `9457` (Problem Details for HTTP APIs)
> - **Compliance & Accessibility**: GDPR (`EU 2016/679`), ePrivacy Directive, European Accessibility Act (`EAA 2025` / WCAG `2.2 AA`)
> - **Web App Manifest**: W3C Web Application Manifest (`manifest.webmanifest`)

---

## Universal Web Essentials Catalogue (Elicit Before Backlog Lock)

During initial project scoping (`init-project`) or pre-launch auditing, walk through all four categories and confirm `MVP Required` vs. `Phase 2` for each item:

### 1. Discovery, SEO & Social Sharing (`role-seo-geo-specialist`)
| Essential Feature | Route / Artifact | Production Engineering Standard |
| :--- | :--- | :--- |
| **Dynamic XML Sitemaps** | `/sitemap.xml` or `/sitemap-index.xml` | Partitioned by entity type (`<= 45k` URLs/file) with accurate DB `<lastmod>` timestamps. |
| **Robots Exclusion File** | `/robots.txt` | Allows CSS/JS assets, blocks `/api/`, `/admin/`, `/auth/`, and links to absolute `Sitemap:` URL. |
| **Canonical & `hreflang` Tags** | `<head>` metadata | Self-referencing `<link rel="canonical">` on every indexable page (including paginated pages). |
| **Dynamic Open Graph Cards** | `/api/og` or static `1200x630` | Custom `1200x630px` social preview images (`og:image`, `twitter:card`) for entities and articles. |
| **Schema.org JSON-LD Graph** | `<script type="application/ld+json">` | Validated `Organization`, `WebSite`, `BreadcrumbList`, and domain entity schemas. |

### 2. Resilience, Error Boundaries & Progressive Web App (`role-frontend-developer`, `design-engineering`)
| Essential Feature | Route / Artifact | Production Engineering Standard |
| :--- | :--- | :--- |
| **Custom 404 Not Found Page** | `404` / `not-found` | Returns true HTTP `404` status, explains clearly, offers search bar and top navigation links back home. |
| **Custom 500 / Global Error Boundary** | `500` / `error` | Catches unexpected rendering/server failures, displays correlation `requestId` and a 1-tap `Try Again` button. |
| **Offline Fallback Page & Banner** | `/offline` + UI banner | Detects lost connectivity (`navigator.onLine`), shows cached data timestamp, and prevents lost form input. |
| **Favicon Suite & Web App Manifest** | `/favicon.ico`, `/icon.svg`, `/apple-touch-icon.png`, `/manifest.webmanifest` | SVG favicon supporting dark/light mode, `180x180` Apple touch icon, `192`/`512` maskable PWA icons, and theme colors. |
| **System Health & Readiness Check** | `/api/health` or `/healthz` | Lightweight endpoint returning HTTP `200 OK` (`{ status: "ok", version, db: "connected" }`) for uptime monitors and zero-downtime deploys. |

### 3. Legal, Privacy & Accessibility Governance (`role-legal-counsel`)
| Essential Feature | Route / Artifact | Production Engineering Standard |
| :--- | :--- | :--- |
| **Privacy Policy & GDPR Notice** | `/privacy` | Declares Data Controller, lawful bases (Art. 6), sub-processors, retention periods, and DSAR/erasure rights. |
| **Terms of Service & Disclaimers** | `/terms` | Defines acceptable use, IP licensing, limitation of liability, and domain-specific disclaimers (medical/financial/civic). |
| **Cookie / Storage Consent Architecture** | Consent Banner / `/cookies` | Cookieless analytics by default (`Umami`), or strict opt-in banner with equal-prominence `Reject All` / `Accept All`. |
| **Accessibility Statement (EAA 2025)** | `/accessibility` | Declares WCAG 2.2 Level AA conformance status, keyboard shortcuts, and accessibility issue reporting contact. |
| **Methodology / Editorial & Data Policy** | `/methodology` or `/about` | Explains how domain data, ratings, prices, clinical guidelines, or public records are sourced, verified, and updated. |

### 4. Trust, Communication & Security Hygiene (`role-security-consultant`, `role-copywriter`)
| Essential Feature | Route / Artifact | Production Engineering Standard |
| :--- | :--- | :--- |
| **Contact, Dispute & Rectification Channel** | `/contact` or `/report-issue` | Rate-limited, spam-protected form (or verified email flow) supporting general inquiries, bug reports, and legal/factual rectification notices. |
| **Vulnerability Disclosure Policy (RFC 9116)** | `/.well-known/security.txt` | Standardized plaintext file specifying `Contact:`, `Expires:` (ISO-8601), `Preferred-Languages: en`, and `Canonical:`. |
| **Public Changelog / Release Notes** | `/changelog` or `/releases` | User-friendly summary of platform improvements, new features, and data updates. |

---

## Mandatory Pre-Launch Web Essentials Audit Checklist

- [ ] **Backlog Coverage**: Every applicable Web Essential above has a dedicated, tracked GitHub Issue in the project backlog prior to Phase 8 feature freeze.
- [ ] **True HTTP Status Codes**: `/non-existent-route` returns HTTP `404` (never a `200 OK` Soft 404); `/api/health` returns `200 OK` when healthy and `503` when dependencies fail.
- [ ] **Footer Navigation Discoverability**: Privacy Policy, Terms of Service, Accessibility Statement, Methodology/About, and Contact links are permanently reachable in the global semantic `<footer>` on every page.
- [ ] **RFC 9116 Validity**: `/.well-known/security.txt` includes a valid future `Expires:` timestamp.

---

## Continuous Evolution & Self-Improvement Mandate
- **Baseline Floor, Not a Ceiling**: Treat this 18-item catalogue as the minimum baseline floor for any public or commercial web platform. Expand it with sector-specific essentials (e.g., `/hipaa-notice` in US Healthcare, `/impressum` in DACH jurisdictions, `/open-data-api` in Civic Tech) inside `<workspace>/.agents/extensions/standards-web-essentials.md`.
