---
kind: playbook
domain: frontend
topics: [frontend, performance]
authority: guidance
---

# Frontend Performance

Performance is a feature. Users abandon pages that load slowly; search engines penalize them. Core Web Vitals are the standard metrics that define "performant" — they are measurable, actionable, and directly correlate with user experience and search ranking.

## Target Metrics

| Metric | Definition | Good threshold |
|---|---|---|
| **LCP** (Largest Contentful Paint) | Time until the largest visible content element is rendered | < 2.5s |
| **CLS** (Cumulative Layout Shift) | Visual stability — how much content moves after initial render | < 0.1 |
| **INP** (Interaction to Next Paint) | Responsiveness — time from user interaction to next frame | < 200ms |
| **FID** (deprecated by INP) | — | — |

These are the Google search ranking signals. Failing all three degrades SEO and user experience simultaneously.

## Optimization Techniques

### Code Splitting

Load only the JavaScript needed for the current page. Everything else loads when needed.

```typescript
// React.lazy for component-level code splitting
import { lazy, Suspense } from 'react';

const InvoiceDetailModal = lazy(() => import('./InvoiceDetailModal'));

function InvoiceList() {
  const [selectedId, setSelectedId] = useState<string | null>(null);
  return (
    <>
      {/* ... list ... */}
      {selectedId && (
        <Suspense fallback={<ModalSkeleton />}>
          <InvoiceDetailModal invoiceId={selectedId} onClose={() => setSelectedId(null)} />
        </Suspense>
      )}
    </>
  );
}
```

Next.js and Vite apply route-level code splitting automatically. Apply component-level splitting for heavy components that are not always rendered (modals, charts, rich text editors).

### Image Optimization

Images are the most common LCP degrader. Use:
- Next.js `<Image>` component (automatic WebP conversion, lazy loading, correct sizing)
- Or explicit `<img loading="lazy" decoding="async">` attributes
- `width` and `height` attributes on every image to prevent CLS

### Font Loading Strategy

Font loading causes layout shift when the fallback font swaps to the web font (FOIT/FOUT).

```typescript
// Next.js font optimization — zero-shift font loading
import { Inter } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',   // show fallback immediately, swap when loaded
  preload: true,
});
```

For non-Next.js: `font-display: swap` in CSS + preload the font file with `<link rel="preload">`.

### Lazy Loading Non-Critical Components

Defer rendering of below-the-fold components until they enter the viewport:

```typescript
// Use Intersection Observer to defer rendering
import { useInView } from 'react-intersection-observer';

function DeferredChart({ data }: { data: ChartData }) {
  const { ref, inView } = useInView({ triggerOnce: true });
  return (
    <div ref={ref}>
      {inView ? <ExpensiveChart data={data} /> : <ChartPlaceholder />}
    </div>
  );
}
```

## Measuring with Lighthouse CI

Run Lighthouse in CI to catch regressions before they ship:

```yaml
# GitHub Actions step
- name: Run Lighthouse CI
  uses: treosh/lighthouse-ci-action@v11
  with:
    urls: |
      https://staging.yourdomain.com/
      https://staging.yourdomain.com/dashboard
    budgetPath: ./lighthouse-budget.json
    temporaryPublicStorage: true
```

`lighthouse-budget.json`:
```json
[
  {
    "path": "/*",
    "timings": [
      { "metric": "largest-contentful-paint", "budget": 2500 },
      { "metric": "cumulative-layout-shift", "budget": 100 },
      { "metric": "interaction-to-next-paint", "budget": 200 }
    ]
  }
]
```

## Related Documents

- [Frontend rendering](rendering.md)
- [Frontend caching](caching.md)
- [Infrastructure CDN and assets](../delivery/infrastructure/cdn-and-assets.md)
- [Non-functional requirements](../product/non-functional-requirements.md)
