---
kind: playbook
domain: operations
topics: [operations, admin, billing, management]
authority: guidance
---

# Admin Billing Management

The billing management panel handles support cases involving subscriptions, plan access, and refunds. All billing operations from the admin panel write to the audit log.

## Stripe Customer Lookup

Every user record should store their Stripe customer ID. Look up billing details via the Stripe API rather than storing redundant billing state in your own database.

```typescript
// GET /admin/users/:userId/billing
app.get('/admin/users/:userId/billing', requireAdmin, async (req, res) => {
  const user = await db.user.findUnique({
    where: { id: req.params.userId },
    select: { stripeCustomerId: true, email: true },
  });

  if (!user?.stripeCustomerId) {
    return res.json({ stripeCustomerId: null, subscriptions: [], invoices: [] });
  }

  const [subscriptions, invoices] = await Promise.all([
    stripe.subscriptions.list({ customer: user.stripeCustomerId, limit: 5 }),
    stripe.invoices.list({ customer: user.stripeCustomerId, limit: 10 }),
  ]);

  res.json({
    stripeCustomerId: user.stripeCustomerId,
    subscriptions: subscriptions.data,
    invoices: invoices.data,
  });
});
```

## Manual Plan Override

For support edge cases — trial extensions, courtesy upgrades, billing errors — allow an admin to manually set a user's subscription tier without going through Stripe's normal flow.

```typescript
// POST /admin/users/:userId/override-plan
app.post('/admin/users/:userId/override-plan', requireAdmin, async (req, res) => {
  const { plan, reason, expiresAt } = z
    .object({
      plan: z.enum(['free', 'pro', 'enterprise']),
      reason: z.string().min(1),
      expiresAt: z.string().datetime().optional(),
    })
    .parse(req.body);

  const adminId = req.session.userId!;

  const before = await db.user.findUnique({
    where: { id: req.params.userId },
    select: { subscriptionPlan: true },
  });

  await db.user.update({
    where: { id: req.params.userId },
    data: {
      subscriptionPlan: plan,
      planOverride: true,
      planOverrideReason: reason,
      planOverrideExpiresAt: expiresAt ? new Date(expiresAt) : null,
    },
  });

  await writeAuditLog({
    actorId: adminId,
    action: 'billing.plan.override',
    targetId: req.params.userId,
    metadata: { before: { plan: before?.subscriptionPlan }, after: { plan }, reason },
  });

  res.json({ success: true });
});
```

## Refund from Admin Panel

```typescript
// POST /admin/users/:userId/refund
app.post('/admin/users/:userId/refund', requireAdmin, async (req, res) => {
  const { invoiceId, amountCents, reason } = z
    .object({
      invoiceId: z.string(),
      amountCents: z.number().int().positive().optional(),
      reason: z.enum(['duplicate', 'fraudulent', 'requested_by_customer']),
    })
    .parse(req.body);

  const adminId = req.session.userId!;

  const invoice = await stripe.invoices.retrieve(invoiceId);
  if (!invoice.charge) {
    return res.status(400).json({ error: 'No charge found for this invoice' });
  }

  const refund = await stripe.refunds.create({
    charge: invoice.charge as string,
    ...(amountCents ? { amount: amountCents } : {}),
    reason,
  });

  await writeAuditLog({
    actorId: adminId,
    action: 'billing.refund.issued',
    targetId: req.params.userId,
    metadata: {
      refundId: refund.id,
      invoiceId,
      amountCents: refund.amount,
      reason,
    },
  });

  res.json({ refundId: refund.id, amount: refund.amount, status: refund.status });
});
```

## Related Documents

- [Admin overview](overview.md)
- [Admin audit log](audit-log.md)
- [Stripe invoices and refunds](../../integrations/providers/stripe/invoices-refunds.md)
- [Payments subscription lifecycle](../../business/payments/subscription-lifecycle.md)
