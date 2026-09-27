---
kind: playbook
domain: quality
topics: [quality, testing, integration]
authority: guidance
---

# Integration Testing

Integration tests verify that multiple components work together correctly — an API route that validates input, queries the database, and returns the right response. They use a real test database, not mocks.

## Test Database Isolation Strategy

Two options for keeping tests isolated:

**Option A: Separate test database with transaction rollback per test (recommended)**

Each test runs in a transaction that is rolled back at the end. The database state is always clean.

```typescript
// tests/setup.ts
import { db } from '../src/lib/db';

beforeEach(async () => {
  await db.$executeRaw`BEGIN`;
});

afterEach(async () => {
  await db.$executeRaw`ROLLBACK`;
});
```

**Option B: Truncate tables between tests**

Slower but more explicit. Useful when your ORM doesn't support raw transactions easily.

```typescript
afterEach(async () => {
  await db.$transaction([
    db.notification.deleteMany(),
    db.oauthAccount.deleteMany(),
    db.user.deleteMany(),
    // ...other tables, respecting FK constraints
  ]);
});
```

Configure a separate test database:

```bash
# .env.test
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/myapp_test
```

```json
// package.json
{
  "scripts": {
    "test": "NODE_ENV=test jest",
    "test:db:setup": "DATABASE_URL=$TEST_DATABASE_URL npx prisma migrate deploy"
  }
}
```

## Fixture and Factory Pattern

Create reusable factory functions for test data:

```typescript
// tests/factories.ts
import { db } from '../src/lib/db';

export async function createUser(overrides: Partial<{
  email: string;
  name: string;
  role: string;
  subscriptionPlan: string;
}> = {}) {
  return db.user.create({
    data: {
      email: overrides.email ?? `test-${Date.now()}@example.com`,
      name: overrides.name ?? 'Test User',
      role: overrides.role ?? 'USER',
      subscriptionPlan: overrides.subscriptionPlan ?? 'free',
    },
  });
}

export async function createAdminUser() {
  return createUser({ role: 'ADMIN', email: 'admin@test.com' });
}
```

## Example Integration Test: API Endpoint

```typescript
// tests/integration/auth.test.ts
import supertest from 'supertest';
import { app } from '../../src/app';
import { createUser } from '../factories';

const agent = supertest.agent(app);

describe('POST /api/auth/login', () => {
  let user: Awaited<ReturnType<typeof createUser>>;

  beforeEach(async () => {
    user = await createUser({ email: 'login-test@example.com' });
    // Set a password hash — adapt to your auth implementation
    await db.user.update({
      where: { id: user.id },
      data: { passwordHash: await bcrypt.hash('correct-password', 10) },
    });
  });

  test('returns 200 and sets session cookie on correct credentials', async () => {
    const res = await agent
      .post('/api/auth/login')
      .send({ email: 'login-test@example.com', password: 'correct-password' })
      .expect(200);

    expect(res.headers['set-cookie']).toBeDefined();
    expect(res.body).toMatchObject({ success: true });
  });

  test('returns AUTH_FAILED with wrong password', async () => {
    const res = await agent
      .post('/api/auth/login')
      .send({ email: 'login-test@example.com', password: 'wrong-password' })
      .expect(401);

    expect(res.body.error).toBe('AUTH_FAILED');
  });

  test('returns AUTH_FAILED with non-existent email — identical to wrong password', async () => {
    const res = await agent
      .post('/api/auth/login')
      .send({ email: 'doesnotexist@example.com', password: 'any-password' })
      .expect(401);

    // Response must be identical whether email doesn't exist or password is wrong
    expect(res.body.error).toBe('AUTH_FAILED');
    // Response body must not distinguish between the two cases
    expect(JSON.stringify(res.body)).not.toContain('email');
    expect(JSON.stringify(res.body)).not.toContain('exist');
  });
});
```

## Related Documents

- [Testing strategy](strategy.md)
- [Testing unit](unit.md)
- [Testing error scenarios](error-scenarios.md)
