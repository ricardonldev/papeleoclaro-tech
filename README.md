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

## Security and testing (added 28 Sep 2026)
- **Security headers** via a Cloudflare `_headers` file: `Strict-Transport-Security`, `X-Content-Type-Options`, `Referrer-Policy`, `X-Frame-Options`, `Permissions-Policy` and `Cross-Origin-Opener-Policy`.
- **Content-Security-Policy in two layers**, because the site runs AdSense and a strict CSP can silently block ads:
  - **Enforced:** blocks plugins, framing by other sites, form posts to other domains, foreign `<base>` tags and anything not served over https. Verified in production: an injected foreign `<base>` was blocked, while AdSense, the consent message, the service worker and the calculators ran with no violations.
  - **Report-Only (strict, Google domains only):** it keeps observing, and the enforced policy will be tightened to that list once served ads show no violations.
- **Tested calculation logic:** the late-filing surcharge and rental deposit calculators were refactored into pure TypeScript modules with **15 tests** (`npm test`, Node's built-in runner). The tests cover tax deadlines per form, full months of delay, the 15 % + late-interest threshold, the 25 % reduction and month-end dates.
- **A real bug found by the tests and fixed:** if the keys were returned on 31 January, the deposit calculator put the interest start date on **3 March** instead of **28 February**, which undercounted the tenant's interest. Now fixed and covered by a test.
- Lighthouse re-run after the change (mobile): 97 home / 96 tool page, and 100 in accessibility, best practices and SEO. Nothing regressed.

## Next improvements
- Tighten the enforced CSP to the Google-only allowlist once served ads show no report-only violations.
- Extend the tests to the remaining calculators.

## Related work
- [ai-app-rescue-case-study](https://github.com/ricardonldev/ai-app-rescue-case-study): securing and shipping an AI-generated Supabase app.
- [whatsapp-cost-calculator](https://github.com/ricardonldev/whatsapp-cost-calculator): pricing rules as tested pure functions ([live](https://ricardonldev.github.io/whatsapp-cost-calculator/)).

---
🇪🇸 **Resumen:** web de calculadoras y guías de trámites en español, hecha con Astro 7, Tailwind 4 y Cloudflare Workers (estática). Lighthouse medido el 28/09/2026: rendimiento de 96-100 y 100 en accesibilidad, buenas prácticas y SEO. Tiene 43 herramientas, parte de ellas generadas a partir de datos; datos estructurados en cada página; y PWA. Añadido el 28/09/2026: cabeceras de seguridad (CSP obligatoria compatible con AdSense y una estricta en observación) y 15 tests en dos calculadoras, que destaparon y permitieron corregir un fallo real de fechas en la de la fianza.
