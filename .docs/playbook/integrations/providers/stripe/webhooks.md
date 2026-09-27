---
kind: playbook
domain: integrations
topics: [integrations, providers, stripe, webhooks]
authority: guidance
---

# Stripe Webhooks

Stripe communicates payment events to your application via webhooks. This document covers the complete event table, signature verification, replay attack prevention, and idempotent event processing.

## Events to Handle

See [payments/subscription-lifecycle.md](../../../business/payments/subscription-lifecycle.md) for the full event → application action mapping. At minimum:

| Event | Handler action |
|---|---|
| `customer.subscription.created` | Set subscription status + provision access |
| `customer.subscription.updated` | Update subscription status, plan, period end |
| `customer.subscription.deleted` | Revoke paid access |
| `customer.subscription.trial_will_end` | Send trial expiry reminder email |
| `invoice.payment_succeeded` | Set status to `active`; send receipt |
| `invoice.payment_failed` | Set status to `past_due`; start dunning |
| `checkout.session.completed` | Grant access for Checkout-completed sessions (one-time purchases) |

## Webhook Handler

```typescript
import express from 'express';
import Stripe from 'stripe';
import { webhookQueue } from '../workers/queues';

// CRITICAL: use raw body — JSON parsing destroys the signature
app.post(
  '/webhooks/stripe',
  express.raw({ type: 'application/json' }),
  async (req, res) => {
    const sig = req.headers['stripe-signature'];

    let event: Stripe.Event;
    try {
      event = stripe.webhooks.constructEvent(
        req.body,
        sig as string,
        process.env.STRIPE_WEBHOOK_SECRET!,
      );
    } catch (err) {
      console.warn('Stripe webhook signature verification failed:', err);
      return res.status(400).send('Webhook error');
    }

    // Enqueue for async processing — respond immediately
    await webhookQueue.add('stripe', {
      eventId: event.id,
      type: event.type,
      data: event.data.object,
    }, {
      jobId: event.id, // Deduplication key — prevents double-processing
    });

    res.json({ received: true });
  }
);
```

## Signature Verification

`stripe.webhooks.constructEvent()` performs two checks:
1. **HMAC-SHA256 signature:** Verifies the payload was sent by Stripe using your webhook signing secret
2. **Timestamp check:** Rejects events with a timestamp more than 5 minutes old, preventing replay attacks

Both checks require the **raw request body** (not parsed JSON). If `express.json()` middleware runs before this route, the body is parsed and the signature check will fail. Route this endpoint with `express.raw()` middleware.

## Idempotent Event Processing

Stripe may send the same event more than once (network issues, retries). Use the event ID as a job ID to prevent processing the same event twice:

```typescript
await webhookQueue.add('stripe', { eventId: event.id, ... }, {
  jobId: event.id, // BullMQ ignores duplicate job IDs
});
```

Also design your handler logic to be safe to run twice:

```typescript
async function handleSubscriptionUpdated(subscription: Stripe.Subscription) {
  await db.user.updateMany({
    where: { stripeSubscriptionId: subscription.id },
    data: { subscriptionStatus: subscription.status },
  });
  // updateMany is idempotent — running it twice produces the same result
}
```

## Security

**Verify the `Stripe-Signature` header on every inbound request before reading the payload.** Without verification, any actor on the internet can POST a fabricated `invoice.payment_succeeded` event to your endpoint and receive paid access. Verification is not optional.

**Reject events outside the 5-minute replay window.** `constructEvent()` handles this automatically. Do not override the tolerance parameter.

**Process events idempotently.** Using the event ID as the job deduplication key prevents double-billing or double-provisioning from Stripe retries.

## Related Documents

- [Payments subscription lifecycle](../../../business/payments/subscription-lifecycle.md)
- [Workers webhooks (general pattern)](../../../backend/async/webhooks.md)
- [Stripe overview](overview.md)
- [Stripe testing](testing.md)
