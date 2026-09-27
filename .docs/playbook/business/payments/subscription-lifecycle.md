---
kind: playbook
domain: business
topics: [business, payments, subscription, lifecycle]
authority: guidance
---

# Subscription Lifecycle

A subscription's state changes in response to events: payment successes, failures, plan changes, cancellations. These state changes arrive as webhook events from the payment provider. The application must handle each event and update the user's access accordingly.

## State Machine

```
Created/Trialing ──────────────────→ Active
       │                               │
       │ (trial ends, no payment)      │ (payment fails)
       ↓                               ↓
  Incomplete                       Past Due
       │                               │
       │ (card added)                  │ (grace period expires)
       └──────────────────→ Active ←──┘│
                                       │ (user cancels)
                                       ↓
                                   Canceled
```

## Webhook Event → Application Action Table

| Stripe Webhook Event | Application Action Required |
|---|---|
| `customer.subscription.created` | Set `subscriptionStatus = 'trialing'` or `'active'`; provision access to paid features |
| `customer.subscription.updated` | Update `subscriptionStatus`, `currentPeriodEnd`, `planId`; adjust feature access if plan changed |
| `customer.subscription.deleted` | Set `subscriptionStatus = 'canceled'`; revoke paid feature access; keep data for a grace period |
| `customer.subscription.trial_will_end` | Send reminder email at 3 days before trial end; prompt user to add payment method |
| `invoice.payment_succeeded` | Set `subscriptionStatus = 'active'`; reset any past-due flags; send payment receipt email |
| `invoice.payment_failed` | Set `subscriptionStatus = 'past_due'`; start dunning sequence; send payment failure email |
| `customer.updated` | Update billing address, default payment method if changed |
| `payment_method.attached` | Optionally notify user that a new payment method was added |

## Implementation

```typescript
// Worker: processStripeWebhook
async function handleStripeEvent(event: Stripe.Event): Promise<void> {
  switch (event.type) {
    case 'customer.subscription.created':
    case 'customer.subscription.updated': {
      const subscription = event.data.object as Stripe.Subscription;
      await db.user.updateMany({
        where: { stripeSubscriptionId: subscription.id },
        data: {
          subscriptionStatus: subscription.status,
          currentPeriodEnd: new Date(subscription.current_period_end * 1000),
          planId: subscription.items.data[0]?.price.id ?? null,
        },
      });
      break;
    }

    case 'customer.subscription.deleted': {
      const subscription = event.data.object as Stripe.Subscription;
      await db.user.updateMany({
        where: { stripeSubscriptionId: subscription.id },
        data: {
          subscriptionStatus: 'canceled',
          stripeSubscriptionId: null,
        },
      });
      break;
    }

    case 'invoice.payment_succeeded': {
      const invoice = event.data.object as Stripe.Invoice;
      if (!invoice.subscription) break;
      await db.user.updateMany({
        where: { stripeSubscriptionId: invoice.subscription as string },
        data: { subscriptionStatus: 'active' },
      });
      break;
    }

    case 'invoice.payment_failed': {
      const invoice = event.data.object as Stripe.Invoice;
      if (!invoice.subscription) break;
      await db.user.updateMany({
        where: { stripeSubscriptionId: invoice.subscription as string },
        data: { subscriptionStatus: 'past_due' },
      });
      await emailQueue.add('dunningEmail', {
        subscriptionId: invoice.subscription,
        invoiceId: invoice.id,
      });
      break;
    }
  }
}
```

## Security

**Webhook signature verification is mandatory.** Process no subscription state changes from unverified webhook payloads. An unverified endpoint accepts arbitrary data from anyone — including fabricated `invoice.payment_succeeded` events for subscriptions that didn't actually pay. Always use `stripe.webhooks.constructEvent()` to verify the signature before processing any event.

See [workers/webhooks.md](../../backend/async/webhooks.md) for the full verification pattern.

## Related Documents

- [Payments provider options](provider-options.md)
- [Dunning](dunning.md)
- [Webhook handling](../../backend/async/webhooks.md)
- [Stripe integration](../../integrations/providers/stripe/overview.md)
