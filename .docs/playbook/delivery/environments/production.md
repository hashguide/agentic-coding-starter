---
kind: playbook
domain: delivery
topics: [delivery, environments, production]
authority: guidance
---

# Production Environment

Production is the only environment where mistakes have real cost. Every production deployment must pass CI and staging first. Changes never go directly to production without passing these gates.

## Pre-Deployment Requirements

Before any code reaches production:

- [ ] All CI checks pass (type check, lint, unit tests, integration tests, build)
- [ ] Change deployed to staging and verified manually
- [ ] Database migration dry-run succeeds against production DB
- [ ] No alerts firing in monitoring before the deploy (don't deploy into an incident)

## Monitoring Must Be Active Before Allowing Traffic

When a new production instance starts, monitoring must be running before users can reach it:

- Uptime check is configured and alert routing is verified (see [observability/uptime.md](../../observability/uptime.md))
- Error monitoring (Sentry or equivalent) is configured with production DSN
- Disk space alerts are configured

Do not launch a production instance and then "set up monitoring later." Later becomes never.

## Deploy Approval Gates

For teams: require a deploy approval step in CI before production. For solo builders: the mental gate is still required — "staging verified, CI green, no active incidents."

```yaml
# .github/workflows/deploy.yml
jobs:
  deploy-production:
    environment: production  # GitHub environment with required reviewers or rules
    needs: [ci, deploy-staging]
    if: github.ref == 'refs/heads/main'
```

## Debugging Production

Production must not be debugged by directly querying live data without an audit trail. Safe debugging practices:

- Read production logs via log aggregation service (never SSH and tail live logs in a long session)
- If database investigation is needed, create a read-only replica query or use a database snapshot
- If impersonating a user for support investigation, use the admin impersonation flow (which logs every action)
- Never apply ad-hoc database changes directly in production — use a migration

## Production Error Handling

Production errors shown to users must be safe, ambiguous, and include a reference ID. Stack traces and internal field names must never appear. See [environments/error-exposure-by-environment.md](error-exposure-by-environment.md) and [backend/error-messages.md](../../backend/runtime/error-messages.md).

## Change Control

For solo builders:
- Every production change goes through a PR and CI
- No direct commits to `main`
- No `git push --force` to `main`

For teams:
- Required CI status checks before merge
- Required reviews before merge to main
- Branch protection rules enforced at the repository level (see [version-control/github-setup.md](../version-control/github-setup.md))

## Related Documents

- [Environments overview](overview.md)
- [Environments staging](staging.md)
- [Environments error exposure by environment](error-exposure-by-environment.md)
- [Infrastructure CI/CD](../infrastructure/ci-cd.md)
- [Observability uptime](../../observability/uptime.md)
