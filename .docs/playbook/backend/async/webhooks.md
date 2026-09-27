---
kind: playbook
domain: backend
topics: [backend, async, webhooks]
authority: guidance
---

# Webhook Handling

An inbound webhook is an HTTP POST from an external service (Stripe, SendGrid, GitHub, Twilio) notifying you that something happened. The handler must respond quickly — most webhook senders time out after 5–30 seconds and will retry if they don't receive a 200. Any real processing must happen asynchronously.

## The Four-Step Pattern

**Step 1: Return 200 immediately.**

The webhook handler's only job is to acknowledge receipt. Do not process the event here.

**Step 2: Verify the signature before trusting the payload.**

Every serious webhook provider includes a signature in the request headers. Verify it before treating the payload as authentic. An unverified webhook endpoint accepts arbitrary data from anyone on the internet — including fabricated "payment succeeded" events.

**Step 3: Enqueue the event as a background job.**

Store the raw event payload (or the verified event object) in the queue for processing.

**Step 4: Process in the worker, not the handler.**

The worker performs the actual business logic: updating database records, sending emails, provisioning access.

## Implementation: Stripe Webhooks

```typescript
import Stripe from 'stripe';
import express from 'express';
import { webhookQueue } from '../workers/queues';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

// IMPORTANT: raw body required for signature verification
// Do NOT use express.json() middleware on this route
app.post(
  '/webhooks/stripe',
  express.raw({ type: 'application/json' }),
  async (req, res) => {
    const sig = req.headers['stripe-signature'];
    if (!sig) return res.status(400).send('Missing stripe-signature header');

    let event: Stripe.Event;
    try {
      // Stripe verifies the signature AND checks the timestamp (replay protection)
      event = stripe.webhooks.constructEvent(
        req.body,
        sig,
        process.env.STRIPE_WEBHOOK_SECRET!,
      );
    } catch (err) {
      // Invalid signature — reject; do not process
      console.warn('Stripe webhook signature verification failed', err);
      return res.status(400).send('Webhook signature verification failed');
    }

    // Step 3: Enqueue for async processing
    await webhookQueue.add('stripe', {
      eventId: event.id,
      type: event.type,
      // Store the full event object for the worker
      data: event.data.object,
    }, {
      jobId: event.id, // Deduplicate: same event ID won't be enqueued twice
      attempts: 3,
      backoff: { type: 'exponential', delay: 2000 },
    });

    // Step 1: Respond immediately
    res.json({ received: true });
  },
);
```

```typescript
// Worker: process Stripe events
import { Worker } from 'bullmq';

const stripeWebhookWorker = new Worker('webhooks', async (job) => {
  if (job.data.type === 'customer.subscription.updated') {
    await handleSubscriptionUpdated(job.data.data);
  } else if (job.data.type === 'invoice.payment_failed') {
    await handlePaymentFailed(job.data.data);
  }
  // Unhandled event types are ignored silently (no throw)
}, { connection });

async function handleSubscriptionUpdated(subscription: Stripe.Subscription) {
  await db.user.update({
    where: { stripeSubscriptionId: subscription.id },
    data: {
      subscriptionStatus: subscription.status,
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
    },
  });
}
```

## Replay Attack Prevention

Stripe's `constructEvent` checks that the webhook timestamp is within 300 seconds of the current time, preventing replay attacks where an attacker captures and re-sends a valid webhook. This protection is built into the SDK — it only works if you pass the raw request body, not the parsed JSON body.

For webhook providers without built-in replay protection, implement it yourself:

```typescript
const WEBHOOK_TOLERANCE_SECONDS = 300;

function verifyTimestamp(timestampHeader: string): void {
  const timestamp = parseInt(timestampHeader, 10);
  const now = Math.floor(Date.now() / 1000);
  if (Math.abs(now - timestamp) > WEBHOOK_TOLERANCE_SECONDS) {
    throw new Error('Webhook timestamp too old — possible replay attack');
  }
}
```

## Security

**An unverified webhook endpoint accepts arbitrary data from anyone on the internet.** Without signature verification, an attacker can POST a fabricated `payment.succeeded` event to your endpoint and receive paid features without paying. Signature verification is not optional — it is the only mechanism that establishes the webhook payload is authentic.

Additionally, use `jobId: event.id` (or an equivalent idempotency key) when enqueuing to ensure that retried webhooks from the provider do not create duplicate jobs.

## Related Documents

- [Workers overview](overview.md)
- [Job patterns (idempotency, retry)](job-patterns.md)
- [Stripe integration](../../integrations/providers/stripe/overview.md)
- [Payments subscription lifecycle](../../business/payments/subscription-lifecycle.md)
