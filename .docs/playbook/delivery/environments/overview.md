---
kind: playbook
domain: delivery
topics: [delivery, environments]
authority: guidance
---

# Environments Overview

Every SaaS product runs in three environments: local development, staging, and production. Each has a distinct purpose, and the consequences of mistakes differ dramatically across them.

## The Three Environments

| Environment | Purpose | Cost of mistakes |
|---|---|---|
| **Local** | Fast iteration with no consequences | Zero — only affects your machine |
| **Staging** | Production-faithful pre-release verification | Low — only affects the team |
| **Production** | Real users, real data, real money | High — directly impacts users and business |

**Local** exists for speed. Tools, overlays, verbose logging, hot reload, seed data — anything that makes development faster is acceptable here, as long as it never ships to production.

**Staging** exists for confidence. It must be production-faithful enough that "it works in staging" means "it will work in production." If staging diverges significantly from production, staging stops being useful.

**Production** is the only environment where mistakes have real cost. Every deployment to production must pass staging, and nothing goes to production without passing CI.

## How This Directory Relates to Other Documents

This directory does not duplicate configuration specifics that belong to their own documents. Instead, it defines environment-specific policy:

- **Log verbosity** — which log level is appropriate per environment. See `logging-by-environment.md`.
- **Error detail** — what an error response reveals per environment. See `error-exposure-by-environment.md`.
- **Feature flags** — which flags are on/off by default per environment. See `feature-flags-by-environment.md`.
- **Secret separation** — each environment has entirely separate credentials. See `secrets-by-environment.md`.

For structured logging format, see [observability/logging.md](../../observability/logging.md). For secrets management patterns, see [security/secrets.md](../../security/secrets.md).

## Related Documents

- [Environments local development](local-development.md)
- [Environments staging](staging.md)
- [Environments production](production.md)
- [Environments parity](environment-parity.md)
