---
kind: playbook
domain: frontend
topics: [frontend, rendering]
authority: guidance
---

# Frontend Rendering Strategies

Rendering strategy determines when and where HTML is generated: at build time, at request time on the server, at request time on the edge, or entirely in the browser. Choosing incorrectly means either paying for server infrastructure you don't need or shipping a product that doesn't rank in search engines.

## Definitions

**CSR (Client-Side Rendering):** The browser downloads a minimal HTML shell and a JavaScript bundle. React renders the UI client-side. No server involvement after the initial page load.

**SSR (Server-Side Rendering):** The server renders HTML on every request and sends it to the browser. JavaScript hydrates the page to make it interactive.

**SSG (Static Site Generation):** HTML is generated at build time and stored as static files. No server rendering at runtime.

**ISR (Incremental Static Regeneration):** Static pages are pre-generated at build time but can be revalidated and regenerated in the background without a full rebuild. Next.js-specific.

**Edge Rendering:** SSR executed at CDN edge nodes close to the user, not in a centralized server region.

## Decision Table

| Rendering mode | Good for | Bad for | SEO | Personalization | Infra complexity |
|---|---|---|---|---|---|
| CSR | Dashboards, authenticated app UI | Landing pages, blogs, any public SEO-critical content | Poor | Excellent | Low |
| SSR | Personalized pages that need SEO (user profiles, product pages) | High-frequency, cheap-to-cache content | Excellent | Excellent | Medium |
| SSG | Marketing pages, blogs, docs, pricing | Frequently-changing data, user-specific content | Excellent | Poor | Low |
| ISR | Marketing pages with moderate update frequency | Real-time data | Excellent | Poor | Low |
| Edge | SSR use cases requiring lowest latency globally | Complex server-side operations (DB queries) | Excellent | Medium | Medium |

## Mixing Strategies Within One App

This is a common fit for products that combine public server-rendered content with interactive application surfaces:

| Section | Strategy | Rationale |
|---|---|---|
| Marketing site (landing, pricing, blog) | SSG or ISR | SEO critical, rarely changes, no personalization |
| Auth pages (login, signup, reset) | SSG + CSR | No SEO needed, minimal server logic |
| Application dashboard | CSR | Personalized, no SEO, fast iteration |
| Public user-facing pages | SSR | SEO needed, personalized |

Next.js makes this mix native: different pages can use different rendering strategies within one project. App Router (Next.js 13+) defaults to Server Components (SSR) and opts into CSR with `'use client'` directives.

## When SSR Is Justified for an App Dashboard

SSR adds server infrastructure cost and complexity. For a standard SaaS dashboard, CSR is sufficient because:
- Users are authenticated — there is no SEO benefit to server-rendering
- Initial load time with a good CSR skeleton (loading states, optimistic UI) is competitive with SSR
- CSR is simpler to deploy and develop

Justify SSR for the dashboard when: the p95 time-to-interactive is measurably worse than acceptable targets with CSR, and profiling confirms the cause is initial JS bundle size rather than a fixable code-splitting issue.

## Related Documents

- [Frontend performance](performance.md)
- [SPA app type](app-types/spa.md)
- [SSR/static app type](app-types/ssr-static.md)
- [Frontend caching](caching.md)
