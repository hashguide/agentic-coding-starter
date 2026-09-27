---
kind: playbook
domain: frontend
topics: [frontend, caching]
authority: guidance
---

# Frontend Caching

Frontend caching uses the browser (and optionally a service worker) to avoid re-fetching resources that haven't changed. The goal is to eliminate unnecessary network requests while ensuring users never see stale content for resources that matter.

## HTTP Cache Header Strategy

Cache headers tell the browser (and intermediate CDN caches) how long to store a resource before re-validating or re-fetching.

| Asset type | Cache-Control directive | Rationale |
|---|---|---|
| Fingerprinted JS/CSS bundles | `public, max-age=31536000, immutable` | Content hash in filename — if name is same, content is same. Cache forever. |
| Fingerprinted images/fonts | `public, max-age=31536000, immutable` | Same as bundles. |
| HTML pages | `public, max-age=0, must-revalidate` | HTML references fingerprinted assets. Short TTL ensures asset updates are picked up. |
| API responses (authenticated) | `private, max-age=0, must-revalidate` | Never cache authenticated responses in a shared cache (CDN). |
| Public API responses | `public, max-age=60, s-maxage=300` | Short TTL for public data; CDN can cache longer. |

**Fingerprinting (content-hash):** Vite, Next.js, and Webpack all include a hash of the file's content in the output filename (`main.a1b2c3d4.js`). When the content changes, the filename changes, and the browser fetches the new file. This allows `max-age=31536000` (1 year) for fingerprinted assets — the URL changing IS the cache invalidation mechanism.

## Service Worker Caching Strategies

Service workers intercept fetch requests and can serve responses from a cache, the network, or a combination.

### Cache-First

Check the cache first; only go to the network if the cache misses. Best for static assets that are already fingerprinted.

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((cached) =>
      cached ?? fetch(event.request).then((response) => {
        const clone = response.clone();
        caches.open('static-v1').then((cache) => cache.put(event.request, clone));
        return response;
      })
    )
  );
});
```

**Use for:** Fingerprinted JS/CSS/image assets.

### Network-First

Go to the network first; fall back to cache if the network fails. Best for HTML pages and API responses where freshness matters.

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    fetch(event.request)
      .then((response) => {
        const clone = response.clone();
        caches.open('pages-v1').then((cache) => cache.put(event.request, clone));
        return response;
      })
      .catch(() => caches.match(event.request))
  );
});
```

**Use for:** HTML pages, critical API data for offline fallback.

### Stale-While-Revalidate

Serve from cache immediately; update the cache in the background. Users see fast responses; cache is kept fresh without blocking rendering.

```javascript
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.open('dynamic-v1').then(async (cache) => {
      const cached = await cache.match(event.request);
      const networkFetch = fetch(event.request).then((response) => {
        cache.put(event.request, response.clone());
        return response;
      });
      return cached ?? networkFetch;
    })
  );
});
```

**Use for:** Non-critical API data, images that don't change frequently.

## Related Documents

- [Frontend performance](performance.md)
- [Architecture backend caching](../backend/architecture/caching.md)
- [Infrastructure CDN and assets](../delivery/infrastructure/cdn-and-assets.md)
- [PWA app type](app-types/pwa.md)
