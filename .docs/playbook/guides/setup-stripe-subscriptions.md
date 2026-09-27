---
kind: playbook
domain: guides
topics: [guides, setup, stripe, subscriptions]
authority: guidance
---

# How to Setup Stripe Subscriptions

**What you'll have at the end:** A working end-to-end subscription flow — user clicks "Upgrade," completes Stripe Checkout, and gains plan access confirmed by a webhook.

---

## Step 1 — Create Product and Price in Stripe Dashboard

1. Stripe Dashboard → **Products** → **Add product**
2. Name the product (e.g., "Pro Plan")
3. Add a price: Recurring, monthly, $29/month
4. Note the **Price ID** (`price_...`) — you'll use this in Step 3
5. Repeat for annual price if offering one

---

## Step 2 — Create a Stripe Customer at Signup

Create a Stripe customer when a user signs up, even before they subscribe. This allows pre-trial billing setup and associates invoices correctly.

```typescript
// In your signup handler
import Stripe from 'stripe';
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

const stripeCustomer = await stripe.customers.create({
  email: user.email,
  metadata: { userId: user.id },
});

await db.user.update({
  where: { id: user.id },
  data: { stripeCustomerId: stripeCustomer.id },
});
```

---

## Step 3 — Create a Checkout Session

```typescript
// POST /api/billing/checkout
app.post('/api/billing/checkout', requireAuth, async (req, res) => {
  const user = await db.user.findUnique({ where: { id: req.session.userId! } });

  const session = await stripe.checkout.sessions.create({
    mode: 'subscription',
    customer: user!.stripeCustomerId!,
    line_items: [
      {
        price: process.env.STRIPE_PRO_PRICE_ID!,
        quantity: 1,
      },
    ],
    success_url: `${process.env.APP_URL}/billing/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.APP_URL}/pricing`,
    subscription_data: {
      trial_period_days: 14, // Remove if no trial
      metadata: { userId: user!.id },
    },
  });

  res.json({ url: session.url });
});
```

On the frontend:

```typescript
async function startCheckout() {
  const { url } = await fetch('/api/billing/checkout', { method: 'POST' }).then(r => r.json());
  window.location.href = url;
}
```

---

## Step 4 — Handle the Webhook to Grant Access

The webhook is the authoritative signal. Do not grant access based on the success URL redirect — the redirect can be faked. The webhook cannot.

```typescript
// POST /api/webhooks/stripe
app.post('/api/webhooks/stripe', express.raw({ type: 'application/json' }), async (req, res) => {
  const sig = req.headers['stripe-signature']!;
  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(req.body, sig, process.env.STRIPE_WEBHOOK_SECRET!);
  } catch {
    return res.status(400).send('Webhook signature verification failed');
  }

  if (event.type === 'customer.subscription.created' || event.type === 'customer.subscription.updated') {
    const sub = event.data.object as Stripe.Subscription;
    const userId = sub.metadata.userId;
    const status = sub.status;

    await db.user.update({
      where: { id: userId },
      data: {
        subscriptionStatus: status,
        subscriptionId: sub.id,
        currentPeriodEnd: new Date(sub.current_period_end * 1000),
        subscriptionPlan: status === 'active' || status === 'trialing' ? 'pro' : 'free',
      },
    });
  }

  if (event.type === 'customer.subscription.deleted') {
    const sub = event.data.object as Stripe.Subscription;
    await db.user.update({
      where: { id: sub.metadata.userId },
      data: { subscriptionStatus: 'canceled', subscriptionPlan: 'free' },
    });
  }

  res.json({ received: true });
});
```

---

## Step 5 — Verify End-to-End with a Test Card

1. Start the Stripe CLI: `stripe listen --forward-to localhost:3000/api/webhooks/stripe`
2. Start your app in development mode
3. Click "Upgrade" → redirected to Stripe Checkout
4. Enter test card `4242 4242 4242 4242`, any future expiry, any CVC
5. Complete checkout
6. Verify: `subscriptionStatus` is `active` or `trialing` in the database
7. Verify: user now has access to Pro features

---

## Related Documents

- [Integrations Stripe subscriptions](../integrations/providers/stripe/subscriptions.md)
- [Integrations Stripe webhooks](../integrations/providers/stripe/webhooks.md)
- [Payments subscription lifecycle](../business/payments/subscription-lifecycle.md)
- [Integrations Stripe testing](../integrations/providers/stripe/testing.md)
