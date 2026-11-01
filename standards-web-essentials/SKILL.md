---
name: standards-web-essentials
description: >-
  Standard boilerplate catalogue and audit checklist of universal website features (Sitemaps, Robots.txt, SEO/Open Graph, 404/500 Error Pages, Cookie Banners, Privacy Policy, Terms of Service, Contact Forms, Security.txt, Web App Manifest/Favicons, and Accessibility Statements) that must be evaluated and elicited prior to backlog finalization.
---

# Universal Web Essentials & Boilerplate Checklist (`standards-web-essentials`)

## Mission
Ensure every web application possesses the fundamental legal, operational, navigational, discovery, and resilience features required of a professional public service. Prevent launching "feature-rich" apps that embarrassingly lack basic 404 handlers, contact channels, sitemaps, or legal disclosures.

---

## Universal Feature Catalogue

During the initial project scoping phase (and prior to feature implementation), the agent MUST propose and confirm which of these boilerplate features are required for the project:

| Category | Component / Feature | Purpose |
|---|---|---|
| **SEO & Crawlability** | **`sitemap.xml`** | Auto-generated XML sitemap index for search engines. |
| | **`robots.txt`** | Directives guiding search bot indexing and crawl rate. |
| | **Dynamic Open Graph / Twitter Cards** | Social share preview cards (1200x630px) for every major entity. |
| | **Canonical URLs** | Meta canonical tags preventing duplicate content penalties. |
| | **Schema.org / JSON-LD** | Semantic structured data (`Person`, `Organization`, `Event`). |
| **Resilience & UX** | **Custom 404 Page (Not Found)** | Friendly, helpful page with search box and link back home. |
| | **Custom 500 Error Page / Error Boundary** | Clean user-facing error state with error ID and retry button. |
| | **Favicon Suite & PWA Manifest** | Multi-resolution icons (16px, 32px, 180px Apple touch, 512px PWA) and `manifest.webmanifest`. |
| | **Offline Fallback Page** | Graceful notification for mobile users who lose connectivity. |
| **Legal & Governance** | **Privacy Policy (`/privacy`)** | Transparent GDPR/compliance disclosure of data controller, rights, and retention. |
| | **Terms of Service (`/terms`)** | Acceptable use, disclaimers, liability limitations, IP rights. |
| | **Cookie / Consent Notice** | Minimal, compliant banner (or cookieless declaration). |
| | **Accessibility Statement (`/accessibility`)** | WCAG 2.2 AA declaration and accessibility feedback contact. |
| | **Editorial Policy & Fact-Check Methodology** | Transparent rubric explaining how data, sources, and claims are audited. |
| **Communication & Security** | **Contact & Correction Form (`/contact`)** | Contact channel with dedicated inquiry or correction report options. |
| | **Security Vulnerability Policy (`/.well-known/security.txt`)** | Standard RFC 9116 contact file for white-hat security researchers. |
| | **Changelog / Release Notes (`/changelog`)** | Public log of major updates, new features, and data refreshes. |
