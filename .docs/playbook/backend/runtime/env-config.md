---
kind: playbook
domain: backend
topics: [backend, runtime, env, config]
authority: guidance
---

# Environment Configuration

Applications fail in confusing ways when required configuration is missing or malformed. A missing database URL produces a runtime error on the first query — not at startup, not with a clear message, but 500ms into the first request with a cryptic stack trace. The fail-fast principle applied to configuration means the app refuses to start if configuration is invalid, and it tells you exactly what is wrong.

## Zod Schema for Environment Validation

Define all required environment variables as a Zod schema. Validate at startup, before any routes are registered or any connections are opened.

```typescript
// src/lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  // Server
  NODE_ENV: z.enum(['development', 'staging', 'production']),
  PORT: z.coerce.number().min(1).max(65535).default(3000),

  // Database
  DATABASE_URL: z.string().url(),

  // Auth
  SESSION_SECRET: z.string().min(32),

  // Stripe
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  STRIPE_WEBHOOK_SECRET: z.string().startsWith('whsec_'),
  STRIPE_PUBLISHABLE_KEY: z.string().startsWith('pk_'),

  // Email
  RESEND_API_KEY: z.string().min(1),
  EMAIL_FROM: z.string().email(),

  // Redis (optional — only required in production)
  REDIS_URL: z.string().url().optional(),

  // App
  APP_URL: z.string().url(),
});

function parseEnv() {
  const result = envSchema.safeParse(process.env);

  if (!result.success) {
    const errors = result.error.flatten().fieldErrors;
    const lines = Object.entries(errors).map(
      ([key, messages]) => `  ${key}: ${messages?.join(', ')}`,
    );

    // Human-readable startup error — never a raw stack trace
    console.error('❌ Invalid environment configuration:\n' + lines.join('\n'));
    console.error('\nCheck your .env file or environment variables and restart.');
    process.exit(1);
  }

  return result.data;
}

export const env = parseEnv();
```

## The Fail-Fast Principle at Startup

`parseEnv()` is called at module load time when `env.ts` is imported. The first import of `env.ts` in the application entry point runs the validation. If it fails, the process exits before any server, route, or connection is started.

```typescript
// src/app.ts — entry point
import './lib/env'; // Validates at import — exits immediately if invalid

import express from 'express';
import { env } from './lib/env';

const app = express();
// ... routes ...

app.listen(env.PORT, () => {
  console.log(`Server running on port ${env.PORT} in ${env.NODE_ENV} mode`);
});
```

**What a clean startup error looks like:**

```
❌ Invalid environment configuration:
  STRIPE_SECRET_KEY: Expected string to start with "sk_"
  RESEND_API_KEY: Required

Check your .env file or environment variables and restart.
```

**What an unhelpful error looks like (what fail-fast prevents):**

```
TypeError: Cannot read properties of undefined (reading 'match')
    at /app/src/integrations/stripe.ts:12:8
    at processTicksAndRejections (node:internal/process/task_queues:95:5)
```

## .env File Conventions

- `.env` — defaults for local development, committed to the repository with no real secrets (use placeholder values)
- `.env.local` — local overrides, never committed (`gitignore`)
- `.env.staging` — staging configuration, never committed
- `.env.production` — production configuration, never committed (use a secrets manager)

```bash
# .gitignore
.env.local
.env.staging
.env.production
```

## Security

Secrets must never appear in source code or logs. The `env.ts` schema validates them from environment variables only — never from hardcoded default values. Providing a real API key as a Zod `.default('sk_real_key')` is a critical mistake — it would commit the key to the repository.

If a secret appears in a log, even briefly, rotate it immediately.

## Related Documents

- [Backend logging](logging.md)
- [Secrets management](../../security/secrets.md)
- [Environments secrets by environment](../../delivery/environments/secrets-by-environment.md)
- [How-to: setup env validation](../../guides/setup-env-validation.md)
