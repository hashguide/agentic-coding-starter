---
kind: playbook
domain: integrations
topics: [integrations, providers, stripe, connect]
authority: guidance
---

# Stripe Connect

Stripe Connect enables marketplace-style platforms where a platform account facilitates payments between buyers and sellers (connected accounts). The platform collects a fee on each transaction; Stripe handles payout to the seller.

## Connected Account Types

| Type | Onboarding | Dashboard access | Best for |
|---|---|---|---|
| **Standard** | Seller creates/connects their own Stripe account | Full Stripe Dashboard | Large marketplaces where sellers are sophisticated businesses |
| **Express** | Stripe-hosted, lightweight onboarding flow | Limited Express Dashboard | Most marketplaces — simplest for sellers, good platform control |
| **Custom** | Fully custom onboarding UI built by the platform | No direct Stripe access | Maximum control; platform owns the entire seller experience |

**Recommendation:** Express for most marketplaces. Standard if your sellers are already Stripe users. Custom only if the UX requirement is so specific that the Express flow cannot be adapted.

## Connected Account Onboarding (Express)

```typescript
// POST /api/connect/onboarding — start Express onboarding
app.post('/api/connect/onboarding', requireAuth, asyncHandler(async (req, res) => {
  const user = await db.user.findUniqueOrThrow({ where: { id: req.userId } });

  // Create or retrieve the connected account
  let connectedAccountId = user.stripeConnectAccountId;
  if (!connectedAccountId) {
    const account = await stripe.accounts.create({ type: 'express' });
    connectedAccountId = account.id;
    await db.user.update({
      where: { id: user.id },
      data: { stripeConnectAccountId: connectedAccountId },
    });
  }

  // Create an onboarding link
  const link = await stripe.accountLinks.create({
    account: connectedAccountId,
    refresh_url: `${process.env.APP_URL}/sellers/onboarding?retry=true`,
    return_url: `${process.env.APP_URL}/sellers/dashboard`,
    type: 'account_onboarding',
  });

  res.json({ url: link.url });
}));
```

## Payment with Platform Fee

When a buyer pays for a seller's product, route the payment through the platform and take a fee:

```typescript
// Create a Payment Intent that routes to the connected account with a platform fee
const paymentIntent = await stripe.paymentIntents.create({
  amount: 5000, // $50.00 in cents
  currency: 'usd',
  application_fee_amount: 500, // $5.00 platform fee (10%)
  transfer_data: {
    destination: seller.stripeConnectAccountId!,
  },
});
```

Stripe automatically deducts the platform fee and transfers the remainder to the connected account on their payout schedule.

## Payout Configuration

Payouts to connected accounts are either:
- **Automatic:** Stripe pays out on a schedule (daily, weekly, monthly) based on the connected account's settings
- **Manual:** Platform triggers payouts via API on a custom schedule

Express accounts default to automatic payouts. Custom accounts require the platform to trigger payouts explicitly.

```typescript
// Trigger a manual payout to a connected account
const payout = await stripe.payouts.create(
  { amount: 4500, currency: 'usd' }, // Amount after Stripe fees
  { stripeAccount: seller.stripeConnectAccountId! },
);
```

## Related Documents

- [Stripe overview](overview.md)
- [Stripe subscriptions](subscriptions.md)
- [Stripe webhooks](webhooks.md)
- [Payments provider options](../../../business/payments/provider-options.md)
