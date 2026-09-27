---
kind: playbook
domain: backend
topics: [backend, architecture, caching]
authority: guidance
---

# Backend Caching

Caching trades consistency for performance: you accept that a reader may see data that is slightly stale in exchange for dramatically faster reads. The decision of where to cache — in-process memory, Redis, HTTP headers, or a CDN edge — depends on the data's read frequency, how stale it can be, and how it's consumed.

## Decision Table

| Data characteristic | Appropriate caching layer | Example |
|---|---|---|
| Frequently read, rarely changes, small | In-process memory (LRU cache) | Feature flag values, config data |
| Frequently read, changes occasionally, shared across instances | Redis | Session data, user subscription status, rate limit counters |
| Frequently read, user-specific, medium staleness tolerance | Redis with per-user key | User's invoice list count, notification badge count |
| API response, same for many users | HTTP Cache-Control headers | Public pricing data, public documentation |
| Static assets | CDN + long TTL | CSS, JS bundles, images |
| Heavy computation result | Redis or materialized view | Aggregate reports, analytics summaries |

## In-Process Memory Cache

Good for data that is the same across all requests, rarely changes, and is fast to compute if the cache is cold.

```typescript
import { LRUCache } from 'lru-cache';

const featureFlagCache = new LRUCache<string, boolean>({
  max: 500,
  ttl: 60_000, // 1 minute TTL
});

async function isFeatureEnabled(flagKey: string, userId: string): Promise<boolean> {
  const cacheKey = `${flagKey}:${userId}`;
  const cached = featureFlagCache.get(cacheKey);
  if (cached !== undefined) return cached;

  const flag = await db.featureFlag.findFirst({ where: { key: flagKey, userId } });
  const value = flag?.enabled ?? false;
  featureFlagCache.set(cacheKey, value);
  return value;
}
```

**Limitation:** In-process caches are not shared across multiple Node.js instances. On a multi-instance deployment, each instance has its own cache — use Redis for shared caches.

## Redis Cache

Good for data shared across application instances, user-specific data, or data that requires explicit invalidation.

```typescript
import { createClient } from 'redis';

const redis = createClient({ url: process.env.REDIS_URL });
await redis.connect();

async function getSubscriptionStatus(userId: string): Promise<'active' | 'free' | 'canceled'> {
  const cacheKey = `subscription:${userId}`;
  const cached = await redis.get(cacheKey);
  if (cached) return cached as 'active' | 'free' | 'canceled';

  const subscription = await db.subscription.findUnique({ where: { userId } });
  const status = subscription?.status ?? 'free';

  await redis.setEx(cacheKey, 300, status); // 5-minute TTL
  return status;
}

// Invalidate on subscription change (called from webhook handler)
async function invalidateSubscriptionCache(userId: string): Promise<void> {
  await redis.del(`subscription:${userId}`);
}
```

## HTTP Cache-Control

For API responses that are the same for multiple users or don't change frequently, let the browser and intermediate caches do the work.

```typescript
// Publicly cacheable response — pricing data
app.get('/api/pricing', (req, res) => {
  res.set('Cache-Control', 'public, max-age=3600, s-maxage=86400');
  res.json(pricingConfig);
});

// Private, short-lived — user-specific data
app.get('/api/invoices', requireAuth, async (req, res) => {
  res.set('Cache-Control', 'private, max-age=0, must-revalidate');
  const invoices = await invoiceService.listForUser(req.userId!);
  res.json(invoices);
});
```

## Cache Invalidation Strategy

Cache invalidation is the hard part. Rules:

1. **TTL-based expiry** is the simplest: data expires after N seconds regardless of whether it changed. Acceptable when staleness within the TTL window is acceptable.
2. **Event-driven invalidation**: when data changes, explicitly delete the cache key. More complex but ensures tighter consistency. Use this for subscription status, user roles, and any data that gates access.
3. **Cache-aside (lazy loading)**: check cache, miss → load from DB → populate cache. The pattern used in all examples above.

## Related Documents

- [Database caching](../../data/caching.md)
- [Frontend caching](../../frontend/caching.md)
- [Infrastructure CDN and assets](../../delivery/infrastructure/cdn-and-assets.md)
