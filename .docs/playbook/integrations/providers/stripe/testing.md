---
kind: playbook
domain: integrations
topics: [integrations, providers, stripe, testing]
authority: guidance
---

# Stripe Testing

Stripe provides a complete test environment with test cards, CLI webhook forwarding, and event simulation. All integration tests should run against test mode credentials.

## Test Card Numbers

| Scenario | Card number | CVC | Expiry |
|---|---|---|---|
| Successful payment | 4242 4242 4242 4242 | Any 3 digits | Any future date |
| Decline (generic) | 4000 0000 0000 0002 | Any | Any |
| Decline (insufficient funds) | 4000 0000 0000 9995 | Any | Any |
| Requires 3D Secure authentication | 4000 0025 0000 3155 | Any | Any |
| 3DS — authentication fails | 4000 0000 0000 9987 | Any | Any |
| Dispute / chargeback | 4000 0000 0000 0259 | Any | Any |

Use these in the Stripe Elements card input or the Checkout form during local testing.

## Stripe CLI — Webhook Forwarding

The Stripe CLI forwards webhook events from Stripe to your local server, enabling local development and testing without a public URL.

```bash
# Install the Stripe CLI
# macOS
brew install stripe/stripe-cli/stripe

# Authenticate
stripe login

# Forward webhooks to local server
stripe listen --forward-to localhost:3000/webhooks/stripe
# Output: Ready! Your webhook signing secret is whsec_test_...
```

The CLI outputs a test webhook signing secret. Add this to `.env.local`:
```
STRIPE_WEBHOOK_SECRET=whsec_test_...  # From stripe listen output
```

## Triggering Test Events

Trigger specific Stripe events from the CLI without completing a real checkout flow:

```bash
# Trigger a payment succeeded event
stripe trigger invoice.payment_succeeded

# Trigger a payment failed event
stripe trigger invoice.payment_failed

# Trigger a subscription deletion
stripe trigger customer.subscription.deleted

# List all available trigger events
stripe trigger --help
```

The triggered event is forwarded to your local server via `stripe listen`.

## Test Clocks

Stripe Test Clocks let you simulate time advancing, which is useful for testing trial expirations and renewal events without waiting for real time to pass.

**Dashboard → Billing → Test clocks → Create test clock**

1. Create a clock with a starting time (e.g., today)
2. Create a customer attached to the clock
3. Create a subscription for the customer with a trial
4. Advance the clock to the trial end date
5. Stripe automatically generates the `customer.subscription.trial_will_end` and subsequent events

## Testing Checkout Locally

Use test card numbers in the Stripe Checkout form on localhost. The checkout form works in test mode on any origin.

## Related Documents

- [Stripe overview](overview.md)
- [Stripe subscriptions](subscriptions.md)
- [Stripe webhooks](webhooks.md)
- [Testing error scenarios](../../../quality/testing/error-scenarios.md)
