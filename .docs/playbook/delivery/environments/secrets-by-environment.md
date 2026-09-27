---
kind: playbook
domain: delivery
topics: [delivery, environments, secrets, by, environment]
authority: guidance
---

# Secrets by Environment

Each environment must use completely separate, non-overlapping credentials. A staging credential compromise must never be able to reach production data or services.

## The Core Rule

**No credential is shared between environments.** This means:

- Separate Stripe account or separate restricted keys per environment
- Separate database (separate host, separate name, separate password)
- Separate email sending domain or sub-account
- Separate OAuth app registrations with separate client IDs and secrets
- Separate API keys for every third-party service

## Why Separation Matters

If staging and production share a Stripe key:
- A staging bug can send charges to real customers
- A compromised staging environment grants the attacker access to production payment data
- A botched staging test can cancel real subscriptions

If staging and production share a database:
- A seed script, migration mistake, or data reset in staging destroys production data
- There is no isolation between "safe to experiment with" and "real user data"

## Credential Matrix

| Service | Local | Staging | Production |
|---|---|---|---|
| Database | `myapp_dev` on local postgres | `myapp_staging` on staging DB host | `myapp_prod` on production DB host |
| Stripe | `sk_test_dev_...` (test mode) | `sk_test_staging_...` (test mode) | `sk_live_prod_...` (live mode) |
| Email (SendGrid) | Console log / no send | `SG.staging_key` (sandbox mode) | `SG.production_key` |
| Twilio | Test credentials | Test credentials | Live credentials |
| OpenAI | Dev API key with $10 limit | Staging API key | Production API key |

## Environment Variable Naming

Prefix environment variable names clearly so it is impossible to accidentally copy a production value to staging:

```bash
# Staging .env
STRIPE_SECRET_KEY=sk_test_staging_xxxx      # "staging" in the name
DATABASE_URL=postgresql://...@staging-db-host.../myapp_staging

# Production .env (different values, different host)
STRIPE_SECRET_KEY=sk_live_xxxx
DATABASE_URL=postgresql://...@prod-db-host.../myapp_prod
```

## Rotating Compromised Credentials

If a staging credential is exposed (committed to git, logged, etc.):
1. Rotate the staging credential immediately (revoke and reissue)
2. Audit that the same credential is not also in use in production
3. Check the audit log for any unexpected activity using the compromised key

Because staging and production use separate keys, rotating a staging key has zero impact on production.

## Security

Staging and production must use completely separate credentials. A staging key compromise must not be a production breach. A shared credential turns a staging security issue into a production security incident.

The separation must extend beyond just the key values — separate accounts where possible so that the access control layer is also separate.

## Related Documents

- [Security secrets](../../security/secrets.md)
- [Environments overview](overview.md)
- [Environments staging](staging.md)
- [Environments local development](local-development.md)
