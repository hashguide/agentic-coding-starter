---
kind: playbook
domain: frontend
topics: [frontend, app, types, client, storage]
authority: guidance
---

# Client Storage Options

Client storage options — localStorage, sessionStorage, IndexedDB, and cookies — differ in capacity, persistence, security profile, and sync characteristics. Choosing the wrong storage type for a given piece of data creates security vulnerabilities or silent data loss.

## Decision Table

| Storage type | Capacity | Persistence | Security sensitivity | Cross-tab sync | Best for |
|---|---|---|---|---|---|
| **localStorage** | ~5–10 MB | Permanent | Low — readable by any JS | Same origin, all tabs | Non-sensitive UI preferences (theme, sidebar state) |
| **sessionStorage** | ~5–10 MB | Session (tab close) | Low — readable by any JS | No — tab-specific | Temporary wizard/flow state within a session |
| **IndexedDB** | ~50% of disk | Permanent | Low — readable by any JS | Yes (with sync logic) | Large amounts of structured data, offline data |
| **HttpOnly Cookies** | ~4 KB | Configurable | High — not readable by JS | Yes — sent on every request | Auth tokens, session identifiers |
| **Regular Cookies** | ~4 KB | Configurable | Low — readable by JS | Yes | Non-sensitive, small values |

## The Auth Token Rule

**Auth tokens belong in HttpOnly cookies. Never in localStorage.**

localStorage is readable by any JavaScript on the page. A successful XSS attack can read every value in localStorage with `Object.entries(localStorage)` and exfiltrate them to an attacker-controlled server. If an auth token is in localStorage, a single XSS vulnerability is a complete authentication bypass.

HttpOnly cookies are inaccessible to JavaScript — not by the application's own code, not by a malicious script. The browser sends them automatically on requests to the same origin.

```typescript
// Server sets the session cookie — HttpOnly, Secure, SameSite=Strict
res.cookie('session', sessionId, {
  httpOnly: true,        // Not readable by JavaScript
  secure: true,          // HTTPS only
  sameSite: 'strict',    // Not sent on cross-site requests (CSRF protection)
  maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
});

// The client never reads this cookie directly — the browser sends it automatically
// Application code never touches document.cookie for auth
```

## What Belongs in localStorage

Non-sensitive UI preferences that improve user experience across sessions:
- Theme (dark/light) — though prefer syncing to a user profile in the database for cross-device consistency
- Sidebar collapsed state
- Table column ordering preferences
- Dismissed announcement banners

Do not store: auth tokens, sensitive personal data, anything that would be useful to an attacker.

## What Belongs in sessionStorage

State that should not persist beyond a browser tab:
- Multi-step form progress where you don't want the user to resume on a different device
- Temporary search parameters within a session

## What Belongs in IndexedDB

Large volumes of structured data:
- Offline app data (local-first architecture — see [local-first.md](local-first.md))
- Cached media files
- Draft content for offline editing

## Security

Auth tokens in localStorage are readable by any JavaScript on the page — always use HttpOnly cookies. This applies to JWT access tokens, refresh tokens, and any other credential that grants access to the user's account.

For applications that mix first-party and third-party JavaScript (analytics, chat widgets, marketing tools), the risk is compounded: a supply chain compromise of any third-party script can exfiltrate localStorage contents across your entire user base.

## Related Documents

- [Auth session management](../../access/authentication/session-management.md)
- [SPA security](spa.md)
- [Security transport](../../security/transport.md)
