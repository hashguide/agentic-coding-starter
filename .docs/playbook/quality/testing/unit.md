---
kind: playbook
domain: quality
topics: [quality, testing, unit]
authority: guidance
---

# Unit Testing

Unit tests validate individual functions and modules in isolation, without touching databases, network, or external services. They run fast (entire suite in seconds) and give immediate feedback during development.

## What Belongs in Unit Tests

**Good candidates:**
- Business logic functions: `calculateProration(plan, daysRemaining)`, `canUserAccessFeature(user, featureKey)`
- Validation schemas: Zod schemas for request bodies, env vars, configuration
- Data transformation utilities: formatting, mapping, normalization
- Error code mapping: `mapErrorCodeToMessage(code)` → safe display strings
- Pure calculation functions: pricing, date arithmetic, permission logic

**Poor candidates:**
- Trivial getters/setters (no logic to test)
- Framework internals (Express routing, Prisma model definitions)
- Database queries (these belong in integration tests — mock-DB tests often diverge from real behavior)
- React components with complex state and user interaction (use integration or E2E)

## Mocking External Dependencies

Unit tests must not call external services. Mock at the module boundary:

```typescript
// src/lib/stripe.ts
export function createStripeCustomer(email: string) {
  return stripe.customers.create({ email });
}

// In tests: mock the module
jest.mock('../lib/stripe', () => ({
  createStripeCustomer: jest.fn().mockResolvedValue({ id: 'cus_mock' }),
}));
```

Mock at the boundary of your code, not deep inside library internals. If you're mocking `stripe.customers.create.mock`, you're testing too deep.

## Example: Testing Business Logic

```typescript
// src/lib/access.ts
export function canAccessFeature(
  plan: 'free' | 'pro' | 'enterprise',
  feature: string
): boolean {
  const planFeatures: Record<string, string[]> = {
    free: ['basic_export', 'up_to_3_projects'],
    pro: ['basic_export', 'up_to_3_projects', 'advanced_analytics', 'api_access'],
    enterprise: ['basic_export', 'up_to_3_projects', 'advanced_analytics', 'api_access', 'sso', 'audit_log'],
  };
  return planFeatures[plan]?.includes(feature) ?? false;
}
```

```typescript
// src/lib/access.test.ts
import { canAccessFeature } from './access';

describe('canAccessFeature', () => {
  test('free plan cannot access advanced_analytics', () => {
    expect(canAccessFeature('free', 'advanced_analytics')).toBe(false);
  });

  test('pro plan can access advanced_analytics', () => {
    expect(canAccessFeature('pro', 'advanced_analytics')).toBe(true);
  });

  test('enterprise plan can access sso', () => {
    expect(canAccessFeature('enterprise', 'sso')).toBe(true);
  });

  test('unknown feature returns false for all plans', () => {
    expect(canAccessFeature('pro', 'nonexistent_feature')).toBe(false);
  });
});
```

## Example: Testing a Validation Schema

```typescript
// src/schemas/signup.ts
import { z } from 'zod';

export const signupSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).max(100),
});
```

```typescript
// src/schemas/signup.test.ts
import { signupSchema } from './signup';

describe('signupSchema', () => {
  test('accepts valid email and password', () => {
    expect(signupSchema.safeParse({ email: 'test@example.com', password: 'secure123' }).success).toBe(true);
  });

  test('rejects invalid email', () => {
    const result = signupSchema.safeParse({ email: 'not-an-email', password: 'secure123' });
    expect(result.success).toBe(false);
  });

  test('rejects password shorter than 8 characters', () => {
    const result = signupSchema.safeParse({ email: 'test@example.com', password: 'short' });
    expect(result.success).toBe(false);
  });
});
```

## Related Documents

- [Testing strategy](strategy.md)
- [Testing integration](integration.md)
