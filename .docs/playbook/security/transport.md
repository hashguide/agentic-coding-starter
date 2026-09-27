---
kind: playbook
domain: security
topics: [security, transport]
authority: guidance
---

# Transport Security

All production traffic must travel over TLS. Unencrypted HTTP exposes session cookies, API tokens, and user data to any observer on the network — including shared Wi-Fi, ISPs, and corporate proxies.

## HTTPS Redirect

Redirect all HTTP traffic to HTTPS at the edge — in your load balancer, reverse proxy, or CDN — before it reaches the application server. Never redirect inside application code; that requires the insecure request to reach your app first.

**Nginx:**
```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}
```

**Caddy** (automatic HTTPS, no explicit redirect needed):
```
example.com {
    # Caddy handles TLS and redirects automatically
    reverse_proxy localhost:3000
}
```

## HTTP Strict Transport Security (HSTS)

HSTS tells browsers to refuse future HTTP connections to your domain for a specified period, preventing protocol downgrade attacks and SSL stripping.

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```

- `max-age=63072000`: two years — browsers will only use HTTPS for this duration
- `includeSubDomains`: applies HSTS to all subdomains (include only if all subdomains serve HTTPS)
- `preload`: opt-in to the browser preload list — means even the first visit uses HTTPS

**Only add `preload` after you are certain every subdomain serves HTTPS.** Removal from the preload list takes months.

## Security Headers

Set these headers on every response. They mitigate XSS, clickjacking, MIME sniffing, and information leakage.

```typescript
import helmet from 'helmet';
import express from 'express';

const app = express();

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'nonce-GENERATED_NONCE'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https://cdn.example.com'],
      connectSrc: ["'self'", 'https://api.example.com'],
      frameAncestors: ["'none'"],
      upgradeInsecureRequests: [],
    },
  },
  hsts: {
    maxAge: 63072000,
    includeSubDomains: true,
    preload: true,
  },
}));
```

| Header | Recommended value | Purpose |
|---|---|---|
| `Content-Security-Policy` | `default-src 'self'; frame-ancestors 'none'` (extend as needed) | Prevent XSS and clickjacking |
| `X-Content-Type-Options` | `nosniff` | Prevent MIME sniffing |
| `X-Frame-Options` | `DENY` | Prevent framing (superseded by CSP `frame-ancestors` but include for older browsers) |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Limit referrer leakage |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=()` | Disable browser features not in use |

## Content Security Policy Notes

A useful CSP requires knowing every external origin your site loads. Start with `default-src 'self'` and expand based on actual requirements. A CSP that allows `'unsafe-inline'` or `'unsafe-eval'` provides limited XSS protection — the point is to eliminate those values, not to allow them as a workaround for inline scripts.

Use nonces for necessary inline scripts rather than `'unsafe-inline'`:

```typescript
import { randomBytes } from 'crypto';

app.use((req, res, next) => {
  res.locals.nonce = randomBytes(16).toString('base64');
  next();
});
```

## Related Documents

- [Security overview](overview.md)
- [Security checklist](checklist.md)
- [Infrastructure hosting options](../delivery/infrastructure/hosting-options.md)
