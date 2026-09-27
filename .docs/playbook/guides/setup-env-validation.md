---
kind: playbook
domain: guides
topics: [guides, setup, env, validation]
authority: guidance
---

# How to Setup Environment Variable Validation

**What you'll have at the end:** Fail-fast startup validation that surfaces missing or malformed environment variables with a clear error message before the application starts accepting requests.

---

## Step 1 — List All Required Environment Variables

Go through your codebase and collect every `process.env.VARIABLE` reference. Group them by category:

```
Database:
  DATABASE_URL

Auth:
  SESSION_SECRET
  SESSION_COOKIE_DOMAIN

Stripe:
  STRIPE_SECRET_KEY
  STRIPE_WEBHOOK_SECRET

Email:
  SENDGRID_API_KEY
  EMAIL_FROM

App:
  APP_URL
  NODE_ENV
  PORT (optional, defaults to 3000)
```

---

## Step 2 — Write the Validation Schema

Use Zod to validate environment variables. This produces readable error messages and type-safe access.

```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  // Database
  DATABASE_URL: z.string().url(),

  // Auth
  SESSION_SECRET: z.string().min(32, 'SESSION_SECRET must be at least 32 characters'),
  SESSION_COOKIE_DOMAIN: z.string().min(1),

  // Stripe
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  STRIPE_WEBHOOK_SECRET: z.string().startsWith('whsec_'),

  // Email
  SENDGRID_API_KEY: z.string().startsWith('SG.'),
  EMAIL_FROM: z.string().email(),

  // App
  APP_URL: z.string().url(),
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development'),
  PORT: z.coerce.number().int().positive().default(3000),
});

function validateEnv() {
  const result = envSchema.safeParse(process.env);

  if (!result.success) {
    const errors = result.error.errors
      .map((e) => `  ${e.path.join('.')}: ${e.message}`)
      .join('\n');

    console.error(`\n[FATAL] Invalid environment variables:\n${errors}\n`);
    process.exit(1);
  }

  return result.data;
}

export const env = validateEnv();
```

---

## Step 3 — Wire Validation to Run at Startup

The validation must run before the server starts handling requests — not lazily when a route first accesses an env var.

```typescript
// server.ts (or index.ts)
import './lib/env';  // Run validation first — process.exit(1) if invalid

import express from 'express';
import { env } from './lib/env';

const app = express();

app.listen(env.PORT, () => {
  console.log(`Server running on port ${env.PORT} (${env.NODE_ENV})`);
});
```

The import of `./lib/env` runs `validateEnv()` immediately. If any variable is missing or malformed, the process exits before the Express server starts.

---

## Step 4 — Use the Validated `env` Object

Replace all `process.env.VARIABLE` accesses in your codebase with the validated `env` object:

```typescript
// Before
const secret = process.env.SESSION_SECRET;

// After (type-safe, validated at startup)
import { env } from '../lib/env';
const secret = env.SESSION_SECRET;
```

The `env` object is typed based on the Zod schema — `env.PORT` is a `number`, not a `string | undefined`.

---

## Step 5 — Provide a Clear Failure Message

Test that the error message is actionable:

```bash
# Remove a required env var and start the server
SESSION_SECRET="" node dist/server.js
```

Expected output:

```
[FATAL] Invalid environment variables:
  SESSION_SECRET: SESSION_SECRET must be at least 32 characters

Process exited with code 1
```

The error must name the specific variable and explain what's wrong — not just "validation failed."

---

## Verify

- [ ] Start the server with a missing required variable — process exits with a clear error
- [ ] Start the server with all variables present — starts normally
- [ ] `env.PORT` is a number (not a string)
- [ ] `env.NODE_ENV` is one of the three allowed values
- [ ] CI pipeline exports all required variables; CI builds pass

---

## Related Documents

- [Security secrets](../security/secrets.md)
- [Environments secrets by environment](../delivery/environments/secrets-by-environment.md)
