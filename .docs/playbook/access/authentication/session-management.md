---
kind: playbook
domain: access
topics: [access, authentication, session, management]
authority: guidance
---

# Session Management

Session management determines how the server proves that subsequent requests come from the same authenticated user who logged in. Two approaches dominate: server-side sessions (stateful) and JWTs (stateless). The choice has significant security implications.

## JWT (Stateless) vs. Server-Side Sessions (Stateful)

| Dimension | JWT | Server-side session |
|---|---|---|
| State location | Token contains claims; no server state needed | Session ID in cookie; state stored server-side (DB/Redis) |
| Revocation | Difficult — token valid until expiry without a denylist | Immediate — delete the session record |
| Scaling | Stateless — any instance verifies without shared state | Requires shared session store (Redis) across instances |
| Logout guarantee | Partial — token valid until expiry unless denylist used | Complete — session deleted on logout |
| Implementation complexity | Medium — correct JWT libraries needed; algorithm confusion attacks | Low — simple DB or Redis lookup |

**Recommendation:** Server-side sessions stored in PostgreSQL or Redis. The revocation argument is decisive: a user who logs out should be immediately unauthenticated. JWTs without a denylist cannot guarantee this.

## Server-Side Session Implementation

```typescript
// Creating a session on login
async function createSession(userId: string, userAgent: string, ip: string): Promise<string> {
  const sessionId = crypto.randomBytes(32).toString('hex'); // 256-bit random token
  const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000); // 7 days

  await db.session.create({
    data: { id: sessionId, userId, userAgent, ip, expiresAt },
  });

  return sessionId;
}

// Setting the session cookie
res.cookie('session', sessionId, {
  httpOnly: true,        // Not readable by JavaScript
  secure: process.env.NODE_ENV === 'production', // HTTPS only in production
  sameSite: 'strict',   // Not sent on cross-origin requests
  maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days in milliseconds
  path: '/',
});

// Validating a session on each request
async function validateSession(sessionId: string): Promise<{ userId: string } | null> {
  const session = await db.session.findUnique({
    where: { id: sessionId },
    select: { userId: true, expiresAt: true },
  });

  if (!session || session.expiresAt < new Date()) {
    if (session) {
      // Clean up expired session
      await db.session.delete({ where: { id: sessionId } });
    }
    return null;
  }

  return { userId: session.userId };
}

// Destroying a session on logout
async function destroySession(sessionId: string): Promise<void> {
  await db.session.delete({ where: { id: sessionId } }).catch(() => {}); // Ignore if already gone
}
```

## Refresh Token Rotation

For long-lived sessions, rotate the session ID (or refresh token) periodically to limit the damage window from a stolen token.

```typescript
// Rotate session ID on privilege escalation (password change, email change)
async function rotateSession(oldSessionId: string, userId: string): Promise<string> {
  const newSessionId = crypto.randomBytes(32).toString('hex');
  const expiresAt = new Date(Date.now() + 7 * 24 * 60 * 60 * 1000);

  await db.$transaction([
    db.session.delete({ where: { id: oldSessionId } }),
    db.session.create({ data: { id: newSessionId, userId, expiresAt } }),
  ]);

  return newSessionId;
}
```

## Security

**HttpOnly, Secure, SameSite=Strict:** The session cookie must have all three attributes. `HttpOnly` prevents JavaScript from reading it (XSS protection). `Secure` ensures it's only sent over HTTPS (interception protection). `SameSite=Strict` prevents the cookie from being sent on cross-site requests (CSRF protection).

**CSRF protection:** `SameSite=Strict` is the primary CSRF defense. For APIs that require cross-origin requests (e.g., an API consumed by a separate frontend domain), use a CSRF token header (e.g., double-submit cookie pattern) because `SameSite=Strict` would block those requests.

**Session invalidation:** When a user changes their password, all sessions except the current one should be destroyed. When a user reports account compromise, all sessions are destroyed.

## Related Documents

- [Auth options](options.md)
- [Auth RBAC](rbac.md)
- [Auth pages layout](../../experience/layouts/auth-pages.md)
- [Client storage](../../frontend/app-types/client-storage.md)
