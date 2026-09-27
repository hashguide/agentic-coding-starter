---
kind: playbook
domain: delivery
topics: [delivery, environments, staging]
authority: guidance
---

# Staging Environment

Staging is the pre-production verification gate. Its purpose is to catch problems that unit and integration tests don't — environment configuration issues, deployment failures, migration problems, and integration edge cases.

## Staging Must Use Real Sandbox Credentials

Staging must use real provider sandbox/test mode credentials, not mocks. This is what separates staging from a test environment:

| Provider | Staging credential type |
|---|---|
| **Stripe** | Test mode keys (`sk_test_...`) |
| **SendGrid** | Real API key with sandbox mode enabled, or a staging-specific account |
| **Twilio** | Test credentials |
| **Database** | Separate staging database — never a copy of production |

Staging credentials must be completely separate from production credentials. A staging credential compromise must not touch production. See [secrets-by-environment.md](secrets-by-environment.md).

## Realistic But Non-Sensitive Seed Data

Staging must have enough data to test all application flows, but must not contain real user data from production. Never copy a production database dump to staging.

Use the same seed script as development, extended with more varied data for staging-specific edge cases:

```bash
# On the staging server, after deployment:
NODE_ENV=staging npx prisma db seed
```

Or use dedicated staging fixtures that represent edge cases (users on different plans, expired subscriptions, incomplete onboarding states) without containing real PII.

## Access Control

Staging should not be publicly accessible or indexable:

**Block indexing:**
```html
<!-- In staging layout -->
<meta name="robots" content="noindex, nofollow">
```

**Optional: IP allowlist or basic auth on staging URL:**
```nginx
# Nginx — restrict staging to your team's IP range or VPN
location / {
    allow 203.0.113.0/24;  # Your office/VPN IP range
    deny all;
    proxy_pass http://localhost:3000;
}
```

This prevents staging from being crawled by search engines or accessed by users who accidentally share a staging URL.

## What Staging Verifies

Each deployment to staging should verify:

- [ ] Application starts and passes the health check (`/api/health`)
- [ ] Database migration applied cleanly
- [ ] Login flow works end-to-end
- [ ] Core workflow (whatever your product's main action is) completes
- [ ] Stripe Checkout and webhook handling work with test cards
- [ ] Email delivery works (check staging inbox or sandbox)

If any of these fail on staging, the deployment must not proceed to production.

## What's Allowed to Differ from Production

| Aspect | Staging | Production |
|---|---|---|
| Scale (server size) | Smaller | Full capacity |
| Payment processing | Stripe test mode | Stripe live mode |
| Email delivery | Sandbox / test inbox | Real delivery |
| Monitoring verbosity | More verbose | Standard |
| Seed data | Fixture data | Real user data |

Everything else — runtime version, database engine, dependency versions, infrastructure topology — must match production. See [environment-parity.md](environment-parity.md).

## Related Documents

- [Environments overview](overview.md)
- [Environments production](production.md)
- [Environments parity](environment-parity.md)
- [Environments secrets by environment](secrets-by-environment.md)
