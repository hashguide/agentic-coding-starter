---
kind: playbook
domain: delivery
topics: [delivery, environments, local, development]
authority: guidance
---

# Local Development Environment

The local environment prioritizes speed. Hot reload, verbose logging, verbose error messages, and permissive configuration are all acceptable here. The only requirement is that nothing designed for local development can leak into production.

## .env.local Conventions

Use a `.env.local` file (or `.env.development.local` in Next.js) for values that override defaults in the local environment:

```bash
# .env.local — never commit this file
NODE_ENV=development
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/myapp_dev
REDIS_URL=redis://localhost:6379

# Stripe test keys
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...  # From `stripe listen` CLI output

# Email — disable real sending locally
EMAIL_PROVIDER=console  # Log emails to console instead of sending

# Auth
SESSION_SECRET=dev-secret-not-for-production-at-all

APP_URL=http://localhost:3000
```

`.env.local` must be in `.gitignore`. Never commit real keys, even test keys, to version control. See [security/secrets.md](../../security/secrets.md).

## Seed Data Strategy

The app should be fully usable locally without real third-party accounts. Create a seed script:

```typescript
// prisma/seed.ts
import { db } from '../src/lib/db';

async function main() {
  // Admin user
  const admin = await db.user.upsert({
    where: { email: 'admin@dev.local' },
    update: {},
    create: {
      email: 'admin@dev.local',
      name: 'Dev Admin',
      role: 'ADMIN',
      subscriptionStatus: 'active',
      subscriptionPlan: 'pro',
    },
  });

  // Regular user
  await db.user.upsert({
    where: { email: 'user@dev.local' },
    update: {},
    create: {
      email: 'user@dev.local',
      name: 'Dev User',
      subscriptionStatus: 'active',
      subscriptionPlan: 'free',
    },
  });

  console.log('Seed complete:', { admin });
}

main();
```

```json
// package.json
{
  "prisma": {
    "seed": "npx ts-node prisma/seed.ts"
  }
}
```

```bash
npx prisma db seed
```

## Mocking External Services

Local development must not send real emails, real SMS messages, or charge real payment methods.

**Email:** Replace the email provider with a console logger or use a local SMTP trap:

```typescript
// lib/email.ts
const sendEmail = process.env.NODE_ENV === 'development'
  ? async (params: EmailParams) => {
      console.log('[DEV EMAIL]', JSON.stringify(params, null, 2));
    }
  : realSendEmail;
```

**Stripe:** Use Stripe test keys (they cannot process real payments) and the Stripe CLI webhook forwarder.

**Twilio SMS:** Use Twilio's test credentials or simply log the SMS to the console.

## Dev-Only Tooling

| Tool | Purpose | Production behavior |
|---|---|---|
| Prisma Studio | Database GUI | Not installed in production |
| Verbose SQL logging | See every query | Log level INFO+ only |
| React error overlay | Full error details in UI | Error boundary — user-friendly message |
| Hot reload | Instant code changes | N/A |
| Seed script | Fixture data | Disabled |

Dev-only tooling must never run in production. Guard with `process.env.NODE_ENV === 'development'` checks or separate dependencies (`devDependencies` vs. `dependencies`).

## Related Documents

- [Environments overview](overview.md)
- [Environments staging](staging.md)
- [Environments secrets by environment](secrets-by-environment.md)
- [Security secrets](../../security/secrets.md)
