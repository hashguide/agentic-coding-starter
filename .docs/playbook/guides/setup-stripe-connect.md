---
kind: playbook
domain: guides
topics: [guides, setup, stripe, connect]
authority: guidance
---

# How to Setup Stripe Connect

**What you'll have at the end:** A working marketplace payment flow where sellers onboard to Stripe, and payments are split between sellers and your platform.

---

## Step 1 — Enable Stripe Connect on Your Platform Account

1. Stripe Dashboard → **Connect** → **Get started**
2. Choose **Express accounts** (recommended for most marketplaces — Stripe hosts the dashboard for sellers)
3. Configure the **Business information** for your platform

---

## Step 2 — Implement Connected Account Onboarding

```typescript
// POST /api/connect/onboard
app.post('/api/connect/onboard', requireAuth, async (req, res) => {
  const user = await db.user.findUnique({ where: { id: req.session.userId! } });

  // Create a Stripe Connect account if the user doesn't have one
  let stripeAccountId = user!.stripeAccountId;

  if (!stripeAccountId) {
    const account = await stripe.accounts.create({ type: 'express' });
    stripeAccountId = account.id;

    await db.user.update({
      where: { id: user!.id },
      data: { stripeAccountId },
    });
  }

  // Generate an onboarding link (expires after one use)
  const accountLink = await stripe.accountLinks.create({
    account: stripeAccountId,
    refresh_url: `${process.env.APP_URL}/seller/onboard/refresh`,
    return_url: `${process.env.APP_URL}/seller/onboard/complete`,
    type: 'account_onboarding',
  });

  res.json({ url: accountLink.url });
});
```

**Handle the return and refresh URLs:**

```typescript
// GET /seller/onboard/complete
// Check whether the account is actually enabled (don't trust the redirect alone)
app.get('/seller/onboard/complete', requireAuth, async (req, res) => {
  const user = await db.user.findUnique({ where: { id: req.session.userId! } });
  const account = await stripe.accounts.retrieve(user!.stripeAccountId!);

  if (account.details_submitted && account.charges_enabled) {
    await db.user.update({
      where: { id: user!.id },
      data: { sellerStatus: 'active' },
    });
    res.redirect('/seller/dashboard');
  } else {
    res.redirect('/seller/onboard/incomplete');
  }
});
```

---

## Step 3 — Create a Test Payment with Platform Fee

```typescript
// POST /api/orders/:orderId/pay
app.post('/api/orders/:orderId/pay', requireAuth, async (req, res) => {
  const order = await db.order.findUnique({
    where: { id: req.params.orderId },
    include: { seller: true },
  });

  if (!order?.seller.stripeAccountId) {
    return res.status(400).json({ error: 'Seller not connected to Stripe' });
  }

  const platformFeePercent = 0.10; // 10% platform fee
  const totalCents = order.amountCents;
  const platformFeeCents = Math.round(totalCents * platformFeePercent);

  const paymentIntent = await stripe.paymentIntents.create({
    amount: totalCents,
    currency: 'usd',
    application_fee_amount: platformFeeCents,
    transfer_data: {
      destination: order.seller.stripeAccountId,
    },
    metadata: { orderId: order.id },
  });

  res.json({ clientSecret: paymentIntent.client_secret });
});
```

---

## Step 4 — Verify Payout Configuration

In the Stripe Dashboard → Connect → your test connected account:
- **Charges enabled:** yes
- **Payouts enabled:** yes
- **Payout schedule:** check whether automatic or manual

For testing, Stripe's Express accounts in test mode can receive test payouts. Trigger a test payout:

```bash
stripe payouts create --amount=1000 --currency=usd --stripe-account=acct_test_...
```

---

## Step 5 — Test End-to-End

1. Create a seller account and complete the Express onboarding flow (use Stripe's test onboarding data)
2. Make a test payment using card `4242 4242 4242 4242`
3. Verify in the Stripe Dashboard that:
   - The charge appears on the platform account
   - The transfer appears on the connected account (minus the platform fee)
4. Check your database that the order is marked paid

---

## Related Documents

- [Integrations Stripe Connect](../integrations/providers/stripe/connect.md)
- [Integrations Stripe testing](../integrations/providers/stripe/testing.md)
- [Payments provider options](../business/payments/provider-options.md)
