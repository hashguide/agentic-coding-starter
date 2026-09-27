---
kind: playbook
domain: architecture
topics: [architecture, patterns, structural, proxy]
authority: guidance
---

# Proxy Pattern

The Proxy pattern controls access to an object by placing an intermediary in front of it. The proxy implements the same interface as the subject and intercepts calls, adding behavior such as caching, lazy initialization, access control, or logging before (or instead of) delegating to the real subject.

## Definition

A proxy is an object that implements the same interface as the subject and intercepts calls to add behavior — caching, access control, lazy loading — before delegating (or not) to the real subject.

## Code Example

A caching proxy in front of a slow external API call:

```typescript
interface ExchangeRateService {
  getRate(from: string, to: string): Promise<number>;
}

// Real implementation — makes an external HTTP call
class LiveExchangeRateService implements ExchangeRateService {
  async getRate(from: string, to: string): Promise<number> {
    const response = await fetch(
      `https://api.exchangerate.host/convert?from=${from}&to=${to}`,
    );
    if (!response.ok) throw new Error(`Exchange rate API error: ${response.status}`);
    const data = await response.json();
    return data.result;
  }
}

// Caching proxy — returns cached values when available
class CachingExchangeRateService implements ExchangeRateService {
  private cache = new Map<string, { rate: number; expiresAt: number }>();
  private readonly ttlMs = 60 * 60 * 1000; // 1 hour

  constructor(private readonly inner: ExchangeRateService) {}

  async getRate(from: string, to: string): Promise<number> {
    const key = `${from}-${to}`;
    const cached = this.cache.get(key);

    if (cached && cached.expiresAt > Date.now()) {
      return cached.rate;
    }

    const rate = await this.inner.getRate(from, to);
    this.cache.set(key, { rate, expiresAt: Date.now() + this.ttlMs });
    return rate;
  }
}

// Compose — callers see ExchangeRateService, don't know about the cache
const exchangeRate: ExchangeRateService = new CachingExchangeRateService(
  new LiveExchangeRateService(),
);

const rate = await exchangeRate.getRate('USD', 'EUR');
```

For server-side caching with Redis rather than in-memory, replace the `Map` with Redis `get`/`set` calls.

## Distinction from Decorator

Both Proxy and Decorator wrap an object and implement the same interface. The conceptual difference:
- **Decorator** adds behavior to enhance the subject (logging, timing, retry)
- **Proxy** controls access to the subject (caching means the subject isn't called at all on a cache hit; access control means the subject isn't called if permission is denied)

In practice, the implementation looks identical. The distinction is semantic — it communicates intent.

## When to Use

- **Caching proxy:** When a call is expensive (external API, heavy computation) and the result can be reused within a time window
- **Access control proxy:** When you want to enforce permissions at the data-access layer rather than the route handler
- **Lazy initialization proxy:** When an object is expensive to create and you want to defer creation until first use

## When to Skip the Proxy

If caching is simple enough to inline (one call, one TTL), use a direct Redis call or a memoize utility rather than a full proxy class. Proxies are worth the ceremony when the logic applies across multiple methods of the same interface.

## Related Documents

- [Decorator pattern](decorator.md)
- [Architecture backend caching](../../../backend/architecture/caching.md)
- [Database caching](../../../data/caching.md)
