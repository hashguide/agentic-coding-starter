---
kind: playbook
domain: security
topics: [security, checklist]
authority: guidance
---

# Security Checklist

Sign off on every item in this checklist before a production launch. Each item references the library document that covers its implementation.

If the product has privileged operators or administrators, require phishing-resistant MFA where possible and document recovery before launch — see [multi-factor authentication](../access/authentication/mfa.md).

## Transport

- [ ] All HTTP traffic redirects to HTTPS at the load balancer or CDN — [transport.md](transport.md)
- [ ] HSTS header is set: `max-age=63072000; includeSubDomains` — [transport.md](transport.md)
- [ ] Content-Security-Policy header is configured with a strict allowlist — [transport.md](transport.md)
- [ ] `X-Content-Type-Options: nosniff` is set — [transport.md](transport.md)
- [ ] `X-Frame-Options: DENY` or CSP `frame-ancestors 'none'` is set — [transport.md](transport.md)
- [ ] `Referrer-Policy: strict-origin-when-cross-origin` is set — [transport.md](transport.md)
- [ ] TLS certificate is valid and auto-renews — [infrastructure/hosting-options.md](../delivery/infrastructure/hosting-options.md)

## Authentication

- [ ] Passwords are hashed with bcrypt (cost factor ≥ 12) — [auth/options.md](../access/authentication/options.md)
- [ ] Session cookies use `HttpOnly`, `Secure`, and `SameSite=Strict` — [auth/session-management.md](../access/authentication/session-management.md)
- [ ] Session is rotated on privilege escalation (login, role change) — [auth/session-management.md](../access/authentication/session-management.md)
- [ ] Login endpoint returns `AUTH_FAILED` for both wrong email and wrong password (no enumeration) — [backend/error-messages.md](../backend/runtime/error-messages.md)
- [ ] Signup endpoint does not confirm whether an email is already registered — [backend/error-messages.md](../backend/runtime/error-messages.md)
- [ ] Login and signup endpoints are rate-limited — [backend/error-messages.md](../backend/runtime/error-messages.md)
- [ ] OAuth state parameter is validated to prevent CSRF — [auth/oauth.md](../access/authentication/oauth.md)
- [ ] OAuth accounts are linked by provider user ID, not email — [auth/oauth.md](../access/authentication/oauth.md)

## Input Validation

- [ ] Every route validates request body/query against a Zod schema — [input-validation.md](input-validation.md)
- [ ] All database queries use ORM or parameterized raw SQL — [input-validation.md](input-validation.md), [database/orm.md](../data/orm.md)
- [ ] No `dangerouslySetInnerHTML` with user-controlled content — [input-validation.md](input-validation.md)
- [ ] File uploads validate MIME type from magic bytes, not Content-Type header — [input-validation.md](input-validation.md)
- [ ] File upload size limits are enforced at the web server layer — [input-validation.md](input-validation.md)
- [ ] Uploaded files are stored outside the web root — [input-validation.md](input-validation.md)

## Secrets

- [ ] No secrets, API keys, or passwords in source code or git history — [secrets.md](secrets.md)
- [ ] `.env` and `*.key` are in `.gitignore` — [secrets.md](secrets.md)
- [ ] All required env vars are validated at startup (fail fast) — [backend/env-config.md](../backend/runtime/env-config.md)
- [ ] Secrets rotation policy is defined and scheduled — [secrets.md](secrets.md)

## Database

- [ ] Row-Level Security is enabled on all tables containing user data — [database/schema-design.md](../data/schema-design.md)
- [ ] All sensitive columns are encrypted at rest — [database/encryption.md](../data/encryption.md)
- [ ] Database connection uses TLS with `rejectUnauthorized: true` — [database/encryption.md](../data/encryption.md)
- [ ] Backups are configured and restore has been tested — [database/backups.md](../data/backups.md)
- [ ] Explicit user erasure requests trigger hard delete / anonymization, not just soft delete — [data-privacy.md](data-privacy.md)

## Frontend

- [ ] Auth tokens are stored in HttpOnly cookies, not localStorage — [architecture/frontend/app-types/client-storage.md](../frontend/app-types/client-storage.md)
- [ ] Content Security Policy blocks inline scripts and untrusted sources — [transport.md](transport.md)
- [ ] No secrets or server-only env vars are in the client bundle — [architecture/frontend/app-types/ssr-static.md](../frontend/app-types/ssr-static.md)
- [ ] Error messages shown to users never expose raw server error details — [design/interaction/error-feedback.md](../experience/interaction/error-feedback.md)

## Infrastructure

- [ ] Firewall rules restrict inbound traffic to required ports only (80, 443) — [infrastructure/vps/system-setup/firewall.md](../delivery/infrastructure/vps/system-setup/firewall.md)
- [ ] SSH is key-based; password authentication is disabled — [infrastructure/vps/system-setup/firewall.md](../delivery/infrastructure/vps/system-setup/firewall.md)
- [ ] Dependency versions are pinned; automated vulnerability scanning is running — [code-quality/dead-code-and-cleanup.md](../quality/code/dead-code-and-cleanup.md)
- [ ] Production access is limited to authorized team members — [admin/user-management.md](../operations/admin/user-management.md)

## Related Documents

- [Security overview](overview.md)
- [Launch checklist](../growth/launch/checklist.md)
- [Compliance overview](../compliance/overview.md)
