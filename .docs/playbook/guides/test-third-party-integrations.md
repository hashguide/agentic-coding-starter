---
kind: playbook
domain: guides
topics: [guides, test, third, party, integrations]
authority: guidance
---

# How to Test Third-Party Integrations

**What you'll have at the end:** A consistent strategy for testing third-party integrations safely, without hitting production services or accumulating real charges.

---

## The Core Decision: Mock vs. Sandbox

Every third-party integration test falls into one of two categories:

**Mock (unit tests):** Replace the provider's SDK with a jest mock. Fast, free, zero network calls. Use for logic that depends on the provider's response but doesn't need to verify the provider's behavior.

**Sandbox/test mode (integration tests):** Use the provider's real test environment with test credentials. Slower and requires network access, but exercises the actual request/response cycle. Use for verifying that your integration handles the provider's real behavior correctly.

| Test type | Use mock | Use sandbox |
|---|---|---|
| Unit test of parsing logic | Yes | No |
| Unit test of a handler that transforms a Stripe event | Yes | No |
| Integration test of full Stripe Checkout flow | No | Yes |
| CI smoke test for every PR | Yes (mocks) | No |
| Pre-release integration verification | No | Yes |

---

## Provider Sandbox Credentials

| Provider | How to get test credentials | Notes |
|---|---|---|
| **Stripe** | Default test mode (sk_test_...) | Use test cards (see [Stripe testing](../integrations/providers/stripe/testing.md)) |
| **SendGrid** | Sandbox mode via `mailSettings.sandboxMode` | Emails don't send; requests succeed |
| **Twilio** | Test credentials (ACtest...) + magic numbers | See [Twilio testing](../integrations/providers/twilio/testing.md) |
| **OpenAI** | No sandbox — mock or use real API with low limits | Set a daily spend limit |
| **Anthropic** | No sandbox — mock or use real API with guardrails | Set per-developer key limits |

---

## Mock Pattern (Unit Tests)

```typescript
// __mocks__/@stripe/stripe-js.ts
jest.mock('stripe', () => {
  return jest.fn().mockImplementation(() => ({
    checkout: {
      sessions: {
        create: jest.fn().mockResolvedValue({ id: 'cs_test_mock', url: 'https://checkout.stripe.com/mock' }),
      },
    },
    customers: {
      create: jest.fn().mockResolvedValue({ id: 'cus_mock' }),
    },
  }));
});
```

---

## Integration Test Checklist

Before merging an integration change, verify:

- [ ] Credentials used in tests are test/sandbox credentials, not production
- [ ] No real money moves (Stripe test cards only)
- [ ] No real emails sent to real users (sandbox mode or test inboxes)
- [ ] No real SMS sent to real phone numbers (Twilio test credentials)
- [ ] Webhook signature validation is tested with correct test secrets
- [ ] Error paths tested: what happens when the provider returns 429, 500, or a network timeout?
- [ ] Tests do not hardcode production API keys or endpoints

---

## Separate Integration Test Suite from Unit Tests

Keep integration tests that require network access in a separate directory or test suite, and run them separately from unit tests:

```json
// package.json
{
  "scripts": {
    "test": "jest --testPathIgnorePatterns=integration",
    "test:integration": "jest --testPathPattern=integration"
  }
}
```

In CI: run unit tests on every PR. Run integration tests on merge to main or on a nightly schedule.

---

## Budget Guardrails for AI Providers

OpenAI and Anthropic have no free sandbox. Protect against runaway costs during testing:

- Create separate API keys for CI and each developer
- Set spending limits on CI keys ($0 — all tests should mock)
- Set per-developer dev key limits ($5–10/month)
- Track token usage in application logs and alert on spikes

---

## Related Documents

- [Stripe testing](../integrations/providers/stripe/testing.md)
- [SendGrid testing](../integrations/providers/sendgrid/testing.md)
- [Twilio testing](../integrations/providers/twilio/testing.md)
- [OpenAI testing](../integrations/providers/openai/testing.md)
- [Anthropic testing](../integrations/providers/anthropic/testing.md)
- [Testing integration tests](../quality/testing/integration.md)
