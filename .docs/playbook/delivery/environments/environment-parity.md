---
kind: playbook
domain: delivery
topics: [delivery, environments, environment, parity]
authority: guidance
---

# Environment Parity

Environment parity means staging behaves like production — same runtime, same database engine, same dependencies. Without parity, "it works in staging" stops being evidence that "it will work in production."

## What Must Match Exactly

| Aspect | Why it matters if it differs |
|---|---|
| **Database engine and version** | Query behavior, index behavior, and data types vary between PostgreSQL versions. An index that works in PG 15 may behave differently in PG 16. |
| **Node.js runtime version** | Module behavior and ES features vary. A package that works in Node 20 may not work in Node 18. |
| **Dependency versions** | Exact same `package-lock.json` or `pnpm-lock.yaml`. Floating versions (`^1.2.0`) mean different packages install at different times. |
| **Infrastructure topology** | App server + database + Redis. If staging runs the DB on the same machine as the app and production uses a separate database server, network latency behavior differs. |
| **Environment variable names** | Missing production env vars that exist in staging fail silently if there's no startup validation. See [how-to/setup-env-validation.md](../../guides/setup-env-validation.md). |
| **Nginx/Caddy config** | Reverse proxy headers, rate limits, and body size limits must match. A request that works locally without a reverse proxy may fail in staging if the headers differ. |

## What Is Allowed to Differ

| Aspect | Staging | Production | Why the difference is acceptable |
|---|---|---|---|
| **Server size** | 1–2 vCPU / 2 GB RAM | Sized for actual load | Cost management; functional behavior is the same |
| **Payment processing** | Stripe test mode | Stripe live mode | No real money; behavior is identical for testing purposes |
| **Email delivery** | Sandbox / test inbox | Real delivery | Prevents accidental emails to real users |
| **Monitoring verbosity** | More verbose | Standard | Extra context for debugging without impacting users |
| **Seed data** | Fixture data | Real user data | Real user data must not leave production |
| **SSL certificate** | Let's Encrypt staging CA | Let's Encrypt production CA | Staging CA certificates are not trusted by browsers — acceptable since staging is team-access-only |

## Detecting Drift

Drift between environments accumulates silently. Periodic audits help:

```bash
# Compare Node.js versions across environments
node --version  # Run on each server

# Compare installed package versions
cat package-lock.json | grep '"node_modules/prisma"' -A 2

# Compare database versions
psql $DATABASE_URL -c "SELECT version();"

# Compare Nginx config checksum (compare staging vs production)
md5sum /etc/nginx/sites-available/app
```

Consider using IaC ([infrastructure/iac.md](../infrastructure/iac.md)) to define both environments from the same configuration, with environment-specific variables as the only differences.

## Related Documents

- [Environments overview](overview.md)
- [Environments staging](staging.md)
- [Environments production](production.md)
- [Infrastructure IaC](../infrastructure/iac.md)
