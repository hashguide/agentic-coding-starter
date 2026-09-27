---
kind: playbook
domain: delivery
topics: [delivery, infrastructure, ci, cd]
authority: guidance
---

# CI/CD Pipeline

A CI/CD pipeline is the gate between writing code and shipping it. Its job is to catch problems before they reach production, and to make deployment a repeatable, low-risk operation.

## CI Pipeline Steps

Every pull request runs this sequence. Nothing merges to main without all steps passing.

| Step | Tool (example) | Fails on |
|---|---|---|
| Type check | `tsc --noEmit` | Type errors |
| Lint | `eslint` | Lint rule violations |
| Unit tests | `jest` | Test failures |
| Integration tests | `jest` + test DB | Test failures, DB connection errors |
| Build | `next build` / `tsc` | Build failures |

```yaml
# .github/workflows/ci.yml
name: CI
on:
  pull_request:
    branches: [main]

jobs:
  ci:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_DB: test_db
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports: ['5432:5432']
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npx prisma migrate deploy
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test_db

      - run: npm run type-check
      - run: npm run lint
      - run: npm test
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test_db
          NODE_ENV: test

      - run: npm run build
```

## Deploy Pipeline Steps

Triggered on merge to main (or on a tagged release). The key principle: run the migration as a dry-run first, then deploy, then verify.

| Step | Purpose |
|---|---|
| **Migration dry-run** | Verify the migration applies cleanly before deploying new code |
| **Build image / artifact** | Produce the deployable artifact |
| **Deploy (rolling or blue-green)** | Bring up new instances before taking down old ones |
| **Health check** | Confirm new instances respond before routing traffic |
| **Smoke test** | Hit a few critical endpoints and assert they return expected responses |
| **Automated rollback trigger** | If error rate spikes above threshold within 5 minutes, roll back automatically |

```yaml
# .github/workflows/deploy.yml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npm run build

      # Migration dry-run against production DB
      - name: Migration dry-run
        run: npx prisma migrate status
        env:
          DATABASE_URL: ${{ secrets.PRODUCTION_DATABASE_URL }}

      # Deploy (example: SSH to VPS and restart via PM2)
      - name: Deploy to VPS
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.VPS_HOST }}
          username: deploy
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /app
            git pull origin main
            npm ci --omit=dev
            npx prisma migrate deploy
            pm2 restart app --update-env

      # Smoke test — verify the deployed app responds
      - name: Smoke test
        run: |
          sleep 10
          curl --fail https://yourdomain.com/api/health
```

## Rollback Strategy

**Zero-downtime rollback options (in order of preference):**

1. **Reverse the deploy** — if using immutable deployments (containers, Vercel, Fly.io), redeploy the previous image/tag
2. **PM2 rollback** — `pm2 reload app` with the previous artifact on a VPS
3. **Database migration rollback** — only feasible if the migration was backward-compatible (additive only); destructive migrations cannot be rolled back cleanly

**Rollback rule:** Backward-compatible migrations (adding a nullable column, adding a new table) allow safe rollback. Destructive migrations (dropping a column, renaming a column, changing a NOT NULL constraint) do not — once deployed, a rollback requires the old code to still work without the column or with the old structure. Design migrations to be backward-compatible first, then backfill later.

## Related Documents

- [VPS zero-downtime deployment](vps/zero-downtime.md)
- [Database migrations](../../data/migrations.md)
- [Environments production](../environments/production.md)
