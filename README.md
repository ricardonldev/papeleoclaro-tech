# papeleoclaro.com: technical overview

**[papeleoclaro.com](https://papeleoclaro.com)** is a Spanish website with free calculators and guides for everyday paperwork: taxes, rent, sick leave, inheritance, fines. I build and run it. This page describes how it's made and how it performs. **Every number below was measured, and the date is given.** The source code is private.

## Performance (Lighthouse 12, measured 28 Sep 2026)

| Page | Device | Performance | Accessibility | Best practices | SEO | LCP | CLS |
|---|---|---|---|---|---|---|---|
| Home | Mobile | **98** | 100 | 100 | 100 | 0.8 s | 0 |
| Home | Desktop | **100** | 100 | 100 | 100 | 0.2 s | 0 |
| Tool page (rent deduction, Basque Country) | Mobile | **96** | 100 | 100 | 100 | 2.2 s | 0 |
| Tool page | Desktop | **100** | 100 | 100 | 100 | 0.8 s | 0.002 |

The HTML is served from Cloudflare's cache (`CF-Cache-Status: HIT`). The home page is about 44 KB of HTML with the CSS inlined, and time to first byte from Madrid was about 60 ms.

## Stack
- **Astro 7** (static output) + **Tailwind CSS 4** + `@astrojs/sitemap`
- **Cloudflare Workers, static assets only**: no server code, custom 404, apex and `www` domains
- **Vanilla JavaScript** for the calculators, with no frontend framework shipped to the browser

## Architecture decisions
- **Data-driven pages.** 43 tool pages. Some are generated from typed data files with `getStaticPaths`: rent deduction per autonomous community, inheritance without a will per family case, late-filing surcharge per tax form. Adding a region or case means adding data, not a new page.
- **CSS inlined at build time** (`inlineStylesheets: 'always'`). The CSS is small, so inlining removes a render-blocking request, which is what keeps mobile LCP low.
- **PWA:** the web manifest and service worker are generated at build time from Astro endpoints (`manifest.webmanifest.ts`, `sw.js.ts`).
- **Privacy by default:** the legal pages that contain the owner's personal data are `noindex` and filtered out of the sitemap.

## Technical SEO
- **Structured data (JSON-LD)** on every tool page: `SoftwareApplication` (with `Offer`), `FAQPage` (question/answer pairs) and `BreadcrumbList`, plus `WebSite` sitewide.
- A **canonical URL** on every page, and a consistent trailing slash (`trailingSlash: 'always'`).
- An **XML sitemap** (46 URLs) and a robots.txt that points to it.
- `lang="es"`, and a mobile-first layout.

## Honest gaps (next improvements)
- **Security headers:** the site doesn't send `Strict-Transport-Security` or a `Content-Security-Policy` yet. Cloudflare Workers static assets support a `_headers` file, so it's a small change.
- **Automated tests** for the calculation logic aren't in place yet. The public projects linked below show how I test that kind of logic.

## Related work
- [ai-app-rescue-case-study](https://github.com/ricardonldev/ai-app-rescue-case-study): securing and shipping an AI-generated Supabase app.
- [whatsapp-cost-calculator](https://github.com/ricardonldev/whatsapp-cost-calculator): pricing rules as tested pure functions ([live](https://ricardonldev.github.io/whatsapp-cost-calculator/)).

---
🇪🇸 **Resumen:** web de calculadoras y guías de trámites en español, hecha con Astro 7, Tailwind 4 y Cloudflare Workers (estática). Lighthouse medido el 28/09/2026: rendimiento de 96-100 y 100 en accesibilidad, buenas prácticas y SEO. Tiene 43 herramientas, parte de ellas generadas a partir de datos; datos estructurados en cada página; y PWA. Pendiente: cabeceras de seguridad y tests automáticos.
