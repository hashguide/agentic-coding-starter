---
kind: playbook
domain: quality
topics: [quality, testing, error, scenarios]
authority: guidance
---

# Error Scenario Testing

Testing failure paths is as important as testing happy paths. The most common security vulnerabilities and UX disasters happen in error paths that were never explicitly tested.

## Failure Scenario Checklist

Every API endpoint and user-facing flow should be tested against these failure modes:

| Scenario | What to test |
|---|---|
| **Network drop mid-request** | Client-side: optimistic state rolls back; user sees actionable error |
| **Server 500** | Response contains no stack trace, no internal field names; reference ID is present |
| **Third-party API down** | App degrades gracefully; returns a clear error, not a hang |
| **Malformed input** | Validation error is returned; no SQL error or crash |
| **Expired session** | Redirect to login with `?next=` param; no raw auth error leaked |
| **Rate limit exceeded** | 429 with retry guidance; no sensitive information in response |
| **File upload too large** | 413 or validation error; no server crash |
| **Duplicate submission** | Idempotency guard prevents double-charge or double-create |

## Auth Error Path Test — Required

Tests must explicitly assert that the login error response is identical whether the email doesn't exist or the password is wrong.

```typescript
// tests/integration/auth-security.test.ts
import supertest from 'supertest';
import { app } from '../../src/app';
import { createUser } from '../factories';
import bcrypt from 'bcrypt';

describe('Login auth enumeration protection', () => {
  beforeEach(async () => {
    await createUser({
      email: 'real-user@example.com',
      passwordHash: await bcrypt.hash('correct-password', 10),
    });
  });

  test('non-existent email returns AUTH_FAILED', async () => {
    const res = await supertest(app)
      .post('/api/auth/login')
      .send({ email: 'nobody@example.com', password: 'any-password' })
      .expect(401);

    expect(res.body.error).toBe('AUTH_FAILED');
  });

  test('wrong password returns AUTH_FAILED', async () => {
    const res = await supertest(app)
      .post('/api/auth/login')
      .send({ email: 'real-user@example.com', password: 'wrong-password' })
      .expect(401);

    expect(res.body.error).toBe('AUTH_FAILED');
  });

  test('non-existent email and wrong password return IDENTICAL response body', async () => {
    const [nonExistentRes, wrongPasswordRes] = await Promise.all([
      supertest(app)
        .post('/api/auth/login')
        .send({ email: 'nobody@example.com', password: 'any-password' }),
      supertest(app)
        .post('/api/auth/login')
        .send({ email: 'real-user@example.com', password: 'wrong-password' }),
    ]);

    // Both return 401
    expect(nonExistentRes.status).toBe(401);
    expect(wrongPasswordRes.status).toBe(401);

    // Both return identical body
    expect(nonExistentRes.body).toEqual(wrongPasswordRes.body);

    // Neither response reveals which field was wrong
    const bodyStr = JSON.stringify(nonExistentRes.body);
    expect(bodyStr).not.toMatch(/email/i);
    expect(bodyStr).not.toMatch(/exist/i);
    expect(bodyStr).not.toMatch(/password/i);
    expect(bodyStr).not.toMatch(/wrong/i);
  });
});
```

## No Sensitive Data in Error Responses

Every error response test should assert that the response is safe:

```typescript
function assertSafeErrorResponse(body: Record<string, unknown>) {
  const bodyStr = JSON.stringify(body);

  // No stack traces
  expect(bodyStr).not.toMatch(/at \w+ \(/);
  expect(bodyStr).not.toMatch(/\.js:\d+:\d+/);

  // No internal field names
  expect(bodyStr).not.toMatch(/passwordHash/);
  expect(bodyStr).not.toMatch(/stripeSecretKey/);
  expect(bodyStr).not.toMatch(/sessionSecret/);

  // Has a reference ID for support
  expect(body.referenceId ?? body.error).toBeDefined();
}

test('server error response is safe', async () => {
  // Simulate a 500 by hitting a route that throws
  const res = await supertest(app).get('/api/test/trigger-error').expect(500);
  assertSafeErrorResponse(res.body);
});
```

## Third-Party API Down

```typescript
test('handles Stripe API unavailability gracefully', async () => {
  // Mock Stripe to throw a network error
  jest.spyOn(stripe.customers, 'create').mockRejectedValue(
    new Error('ECONNREFUSED')
  );

  const res = await supertest(app)
    .post('/api/billing/checkout')
    .set('Cookie', authCookie)
    .expect(503);

  // Should return a safe, user-facing message
  expect(res.body.error).toBeDefined();
  expect(JSON.stringify(res.body)).not.toContain('ECONNREFUSED');
  expect(JSON.stringify(res.body)).not.toContain('stripe');
});
```

## Security

Auth error-path tests must explicitly assert that the error response is identical whether the email doesn't exist or the password is wrong. An implementation that passes the happy path but fails this test is a user enumeration vulnerability.

## Related Documents

- [Testing strategy](strategy.md)
- [Testing integration](integration.md)
- [Backend error messages](../../backend/runtime/error-messages.md)
- [Design interaction error-feedback](../../experience/interaction/error-feedback.md)
