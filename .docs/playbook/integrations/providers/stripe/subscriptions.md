---
kind: playbook
domain: integrations
topics: [integrations, providers, stripe, subscriptions]
authority: guidance
---

# Stripe Subscriptions

This document covers creating Products and Prices in Stripe, creating customers on signup, and initiating a subscription via Checkout Session.

## Create a Product and Price

Products and prices can be created in the Stripe Dashboard or via API. For simple setups, the dashboard is sufficient; for programmatic control (e.g., seeding test environments), use the API.

**Dashboard:** Products → Add product → Set name, description, pricing model (recurring), billing interval, price.

**API:**

```typescript
// Run once to create the product and price; save the price ID to env
const product = await stripe.products.create({
  name: 'Pro Plan',
  description: 'Full access to all features',
});

const price = await stripe.prices.create({
  product: product.id,
  unit_amount: 2900, // $29.00 in cents
  currency: 'usd',
  recurring: { interval: 'month' },
});

console.log('Price ID:', price.id); // Save as STRIPE_PRICE_ID
```

## Create a Customer on Signup

Every user who might subscribe should have a Stripe Customer ID. Create it at signup, not at the moment of purchase — this enables pre-purchase customer context (trial tracking, promotional pricing).

```typescript
async function createStripeCustomer(user: { id: string; email: string; name: string }) {
  const customer = await stripe.customers.create({
    email: user.email,
    name: user.name,
    metadata: { userId: user.id },
  });

  await db.user.update({
    where: { id: user.id },
    data: { stripeCustomerId: customer.id },
  });

  return customer;
}
```

## Checkout Session

Use Stripe Checkout for payment — it handles payment method collection, SCA compliance, and receipt sending. The app does not handle card details.

```typescript
// POST /api/checkout — create a Checkout Session
app.post('/api/checkout', requireAuth, asyncHandler(async (req, res) => {
  const user = await db.user.findUniqueOrThrow({ where: { id: req.userId } });

  const session = await stripe.checkout.sessions.create({
    customer: user.stripeCustomerId!,
    payment_method_types: ['card'],
    mode: 'subscription',
    line_items: [{ price: process.env.STRIPE_PRICE_ID!, quantity: 1 }],
    success_url: `${process.env.APP_URL}/dashboard?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.APP_URL}/pricing`,
    subscription_data: {
      trial_period_days: 14,
      metadata: { userId: user.id },
    },
  });

  res.json({ url: session.url });
}));
```

```tsx
// Client: redirect to Checkout
async function handleSubscribe() {
  const { url } = await fetch('/api/checkout', { method: 'POST' }).then(r => r.json());
  window.location.href = url;
}
```

## Billing Portal

After subscription, users manage their billing via the Stripe Customer Portal — update card, view invoices, cancel subscription.

```typescript
app.post('/api/billing-portal', requireAuth, asyncHandler(async (req, res) => {
  const user = await db.user.findUniqueOrThrow({ where: { id: req.userId } });

  const session = await stripe.billingPortal.sessions.create({
    customer: user.stripeCustomerId!,
    return_url: `${process.env.APP_URL}/settings/billing`,
  });

  res.json({ url: session.url });
}));
```

## Related Documents

- [Stripe overview](overview.md)
- [Stripe webhooks](webhooks.md)
- [Stripe testing](testing.md)
- [Payments subscription lifecycle](../../../business/payments/subscription-lifecycle.md)
