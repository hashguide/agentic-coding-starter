---
kind: playbook
domain: growth
topics: [growth, marketing, seo]
authority: guidance
---

# SEO

Search engine optimization makes the product discoverable to users who are searching for the problem it solves. Technical SEO ensures search engines can crawl and index the site correctly; content SEO attracts visitors through relevant queries; Core Web Vitals affect ranking directly.

## On-Page SEO Checklist

For every page:

- [ ] **Unique title tag:** Describes the page specifically. Format: `[Page topic] | [Product name]`. Max 60 characters. Every page has a different title.
- [ ] **Unique meta description:** 120–160 characters. Summarizes the page and includes the primary keyword naturally. Does not duplicate any other page's description.
- [ ] **Logical heading hierarchy:** One `<h1>` per page, matching the topic. `<h2>` for major sections. `<h3>` for subsections. Do not skip heading levels.
- [ ] **Descriptive alt text on all images:** Alt text describes what the image shows for screen readers and image search. Do not use "image" or "photo of" — just describe the content. Decorative images get `alt=""`.
- [ ] **sitemap.xml:** Lists all public, indexable URLs. Submit to Google Search Console and Bing Webmaster Tools.
- [ ] **robots.txt:** Present and configured to allow indexing of public pages. Disallow authenticated / private routes.

```
# robots.txt
User-agent: *
Disallow: /api/
Disallow: /dashboard/
Disallow: /settings/
Allow: /

Sitemap: https://yourdomain.com/sitemap.xml
```

## Core Web Vitals as a Ranking Signal

Google uses Core Web Vitals as a direct ranking signal:

| Metric | Threshold (Good) | Optimization target |
|---|---|---|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | Optimize the hero image/video; preload critical resources |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | Reserve space for images and ads; avoid dynamically injected content above the fold |
| **INP** (Interaction to Next Paint) | ≤ 200ms | Reduce JavaScript execution time; defer non-critical scripts |

A page that ranks #3 with excellent Core Web Vitals may outrank a page at #1 with poor ones, all else being equal. Performance is not separate from SEO — they are the same optimization.

Measure with Lighthouse CI in your deployment pipeline and with Google Search Console's Core Web Vitals report for real-user data.

## Content SEO

Pages that target specific search queries must match the search intent:

| Query type | Intent | Content type |
|---|---|---|
| "best invoicing software for freelancers" | Comparison / evaluation | Comparison page or category landing page |
| "how to send invoices automatically" | How-to | Tutorial or guide |
| "Invoice Pilot vs Bonsai" | Direct comparison | Comparison page targeting both brand names |
| "invoice reminder email template" | Resource / download | Template page |

**Comparison pages and tutorials rank faster** than general category pages because they have more specific, lower-competition search intent.

## Internal Linking

Link related pages to each other using descriptive anchor text. Internal links distribute link equity and help search engines understand content relationships. Use the linked page's primary keyword as the anchor text (not "click here" or "learn more").

## Related Documents

- [AEO — AI answer engine visibility](aeo.md)
- [Content marketing](content-marketing.md)
- [Frontend performance](../../frontend/performance.md)
- [Landing page layout](../../experience/layouts/landing-page.md)
