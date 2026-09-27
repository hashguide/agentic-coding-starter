---
kind: playbook
domain: frontend
topics: [frontend, app, types, server, storage]
authority: guidance
---

# Server-Side Storage

Server-side storage covers data that the server manages on behalf of the user session or request — server-side sessions, key-value stores (Redis), and CDN edge caching for server-rendered content. These complement client-side storage by keeping sensitive or high-frequency data on infrastructure you control.

## Server-Side Sessions vs. Stateless JWTs

Both approaches authenticate users on subsequent requests after login. The choice is between keeping state on the server (session) or encoding it in a signed token (JWT).

| Dimension | Server-side session | Stateless JWT |
|---|---|---|
| Revocation | Immediate — delete the session record | Delayed — JWT valid until expiry; requires a denylist for immediate revocation |
| Scaling | Requires shared session store (Redis) across instances | Stateless — any instance can verify without a shared store |
| Payload | Opaque ID; data fetched from store | Contains claims directly; reduces DB lookups |
| Security on logout | Guaranteed — session deleted | Partial — JWT remains valid until expiry unless denylist used |
| Complexity | Simple | Complex once revocation is needed |

**Recommendation:** For most SaaS products, server-side sessions stored in Redis are simpler and safer than JWTs. The scaling argument for JWTs is only relevant at a scale where Redis becomes a bottleneck — not at typical SaaS launch scale.

```typescript
// Creating a server-side session
async function createSession(userId: string): Promise<string> {
  const sessionId = crypto.randomUUID();
  const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000); // 7 days

  await db.session.create({
    data: { id: sessionId, userId, expiresAt },
  });

  return sessionId;
}

// Verifying a session
async function validateSession(sessionId: string): Promise<{ userId: string } | null> {
  const session = await db.session.findUnique({ where: { id: sessionId } });
  if (!session || session.expiresAt < new Date()) return null;
  return { userId: session.userId };
}
```

## KV Store Use Cases (Redis)

Redis provides fast, in-memory key-value storage for data that doesn't need relational querying:

| Use case | Redis key pattern | TTL strategy |
|---|---|---|
| Rate limiting counters | `rate:${userId}:${window}` | TTL = window size (e.g., 60s) |
| Session store | `session:${sessionId}` | TTL = session expiry |
| Feature flag cache | `flag:${flagKey}:${userId}` | TTL = 60s; invalidate on flag change |
| Background job deduplication | `job-dedup:${jobKey}` | TTL = job processing window |
| Subscription status cache | `subscription:${userId}` | TTL = 5min; invalidate on webhook |

## CDN Edge Caching for Server-Rendered Content

Server-rendered HTML can be cached at the CDN edge so that most requests are served without hitting the origin server. This requires:

1. **Correct `Cache-Control` headers:** The server must signal that the response is cacheable.
2. **Cache key strategy:** Include variation dimensions (locale, A/B variant) in the cache key to avoid serving the wrong variant.
3. **Purge mechanism:** The CDN must be purgeable when content changes (Cloudflare Cache API, Fastly instant purge, etc.).

```typescript
// Express — cacheable server-rendered response
app.get('/blog/:slug', async (req, res) => {
  const post = await fetchPost(req.params.slug);
  if (!post) return res.status(404).send('Not found');

  res.set({
    'Cache-Control': 'public, max-age=60, s-maxage=3600',
    'Surrogate-Key': `post:${post.id}`, // Cloudflare/Fastly tag for selective purge
    'Vary': 'Accept-Encoding',
  });

  res.send(renderBlogPost(post));
});
```

For authenticated content: always set `Cache-Control: private` or `no-store` — CDNs must never cache responses that contain user-specific data.

## Related Documents

- [Auth session management](../../access/authentication/session-management.md)
- [Backend caching](../../backend/architecture/caching.md)
- [Infrastructure CDN and assets](../../delivery/infrastructure/cdn-and-assets.md)
- [Client storage](client-storage.md)
