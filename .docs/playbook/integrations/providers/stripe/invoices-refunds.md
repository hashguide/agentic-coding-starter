---
kind: playbook
domain: integrations
topics: [integrations, providers, stripe, invoices, refunds]
authority: guidance
---

# Stripe Invoices, Refunds, and Cancellations

This document covers day-to-day billing operations: refunding a charge, canceling a subscription, and understanding how Stripe handles proration on mid-cycle plan changes.

## Issuing a Refund

Refund a specific charge by charge ID (found on the Stripe payment object or in the dashboard):

```typescript
async function issueRefund(params: {
  chargeId: string;
  amount?: number; // Optional: partial refund in cents. Omit for full refund.
  reason?: 'duplicate' | 'fraudulent' | 'requested_by_customer';
}): Promise<Stripe.Refund> {
  const refund = await stripe.refunds.create({
    charge: params.chargeId,
    amount: params.amount, // Omit for full refund
    reason: params.reason ?? 'requested_by_customer',
  });

  // Log the refund for audit trail
  await db.auditLog.create({
    data: {
      action: 'refund_issued',
      resourceId: params.chargeId,
      metadata: { refundId: refund.id, amount: refund.amount },
    },
  });

  return refund;
}
```

Refunds appear on the customer's card within 5–10 business days depending on their bank.

## Canceling a Subscription

Two cancellation modes:

### Cancel Immediately

The subscription ends now. The customer loses access immediately. No proration refund is issued automatically (you can issue one manually).

```typescript
await stripe.subscriptions.cancel(subscriptionId);
```

### Cancel at Period End

The subscription continues until the current billing period ends, then cancels. This is the standard "cancel anytime" behavior — customers retain access through their paid period.

```typescript
await stripe.subscriptions.update(subscriptionId, {
  cancel_at_period_end: true,
});
```

The webhook event `customer.subscription.updated` fires with `cancel_at_period_end: true`. When the period ends, `customer.subscription.deleted` fires.

To allow the customer to reverse a "cancel at period end" decision:

```typescript
await stripe.subscriptions.update(subscriptionId, {
  cancel_at_period_end: false,
});
```

## Plan Changes and Proration

When a customer upgrades or downgrades mid-cycle, Stripe calculates a proration credit/charge:

- **Upgrade:** Customer is charged the prorated difference for the remaining days on the old plan. For example, upgrading from $29/mo to $99/mo with 15 days left in the billing cycle creates a prorated charge of ~$35.
- **Downgrade:** Customer receives a prorated credit toward the next invoice.

```typescript
// Upgrade a subscription to a new price
async function changeSubscriptionPlan(subscriptionId: string, newPriceId: string) {
  const subscription = await stripe.subscriptions.retrieve(subscriptionId);
  const subscriptionItemId = subscription.items.data[0].id;

  await stripe.subscriptions.update(subscriptionId, {
    items: [{ id: subscriptionItemId, price: newPriceId }],
    proration_behavior: 'create_prorations', // Default — creates a proration invoice item
  });
}
```

**`proration_behavior` options:**
- `create_prorations` (default): Stripe creates a proration invoice item; customer is charged/credited on the next invoice
- `always_invoice`: Stripe immediately invoices the proration difference
- `none`: No proration — customer simply switches to the new plan on the next billing date

## Related Documents

- [Stripe subscriptions](subscriptions.md)
- [Stripe webhooks](webhooks.md)
- [Payments dunning](../../../business/payments/dunning.md)
- [Admin billing management](../../../operations/admin/billing-management.md)
