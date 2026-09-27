---
kind: playbook
domain: frontend
topics: [frontend, app, types, pwa]
authority: guidance
---

# Progressive Web App (PWA)

A Progressive Web App is a web application that uses service workers, a web app manifest, and modern browser APIs to provide an app-like experience: offline capability, installability, push notifications, and fast load times. PWAs are appropriate when offline functionality or app-store-like installation is a genuine product requirement.

## Service Worker Setup

The service worker is the core of a PWA. It intercepts network requests, manages caches, and enables offline functionality.

```typescript
// public/service-worker.ts (compiled and served as service-worker.js)
const CACHE_NAME = 'app-v1';
const PRECACHE_URLS = [
  '/',
  '/offline.html',
  '/static/js/main.js',
  '/static/css/main.css',
];

// Install: precache essential assets
self.addEventListener('install', (event: ExtendableEvent) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => cache.addAll(PRECACHE_URLS)),
  );
  (self as ServiceWorkerGlobalScope).skipWaiting();
});

// Activate: clean up old caches
self.addEventListener('activate', (event: ExtendableEvent) => {
  event.waitUntil(
    caches.keys().then((keys) =>
      Promise.all(keys.filter((k) => k !== CACHE_NAME).map((k) => caches.delete(k))),
    ),
  );
});

// Fetch: network-first with offline fallback
self.addEventListener('fetch', (event: FetchEvent) => {
  if (event.request.mode === 'navigate') {
    event.respondWith(
      fetch(event.request).catch(() => caches.match('/offline.html') as Promise<Response>),
    );
    return;
  }
  // Cache-first for static assets
  event.respondWith(
    caches.match(event.request).then((cached) => cached ?? fetch(event.request)),
  );
});
```

Register the service worker in the app entry point:

```typescript
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/service-worker.js');
  });
}
```

## Offline Capability Strategy

Define what the app does when offline before building it:

| Scenario | Strategy |
|---|---|
| Read previously-loaded data | Cache API responses; serve from cache when offline |
| Create new content offline | Store in IndexedDB; sync to server on reconnect |
| No connection at all | Show offline.html explaining limited functionality |

Use `navigator.onLine` and the `online`/`offline` events to communicate connectivity state to the user.

## Web App Manifest

```json
{
  "name": "InvoiceApp",
  "short_name": "InvoiceApp",
  "description": "Send invoices from anywhere",
  "start_url": "/dashboard",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#2563eb",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

## Install Prompt UX

The browser's native install prompt appears when PWA criteria are met. You can capture and defer it to show at a more appropriate moment:

```typescript
let deferredPrompt: BeforeInstallPromptEvent | null = null;

window.addEventListener('beforeinstallprompt', (e) => {
  e.preventDefault();
  deferredPrompt = e as BeforeInstallPromptEvent;
  // Show your custom install button
  setShowInstallBanner(true);
});

async function handleInstallClick() {
  if (!deferredPrompt) return;
  deferredPrompt.prompt();
  const { outcome } = await deferredPrompt.userChoice;
  deferredPrompt = null;
  setShowInstallBanner(false);
}
```

## Security

**Service worker scope must be tightly defined.** A service worker registered at `/` intercepts all requests on the origin. Register from the app's root only if you intend to intercept all requests.

**Cache poisoning risk:** If the service worker fetches resources from untrusted origins and caches the response, an attacker who compromises those origins can poison the cache with malicious content that persists offline. Precache only your own origin's assets. Verify the integrity of cached responses if third-party content must be cached.

## Related Documents

- [Frontend caching](../caching.md)
- [Client storage](client-storage.md)
- [Local-first architecture](local-first.md)
