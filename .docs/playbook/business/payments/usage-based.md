---
kind: playbook
domain: business
topics: [business, payments, usage, based]
authority: guidance
---

# Usage-Based Billing Implementation

Usage-based billing requires metering usage in your system, reconciling with the payment provider, and surfacing current usage to the user in the product. The billing page is a first-class feature — users managing their costs need accurate, timely information.

## Metering in Your Own Database

Record every usage event in your own database. This is the source of truth for disputes and for your in-product usage dashboard. The billing provider's records are for billing; your records are for everything else.

```typescript
// Schema
model UsageEvent {
  id          String   @id @default(uuid())
  userId      String
  type        String   // 'email_sent' | 'api_request' | 'token_generated'
  quantity    Int      @default(1)
  metadata    Json?
  recordedAt  DateTime @default(now())
  reportedAt  DateTime? // Null until reported to billing provider

  @@index([userId, type, recordedAt])
}
```

```typescript
// Usage recording (called at the point of consumption)
async function consumeUsage(params: {
  userId: string;
  type: string;
  quantity?: number;
  metadata?: Record<string, unknown>;
}): Promise<void> {
  const event = await db.usageEvent.create({
    data: {
      userId: params.userId,
      type: params.type,
      quantity: params.quantity ?? 1,
      metadata: params.metadata,
      recordedAt: new Date(),
    },
  });

  // Immediately report to Stripe (or batch report in a scheduled job)
  try {
    const user = await db.user.findUniqueOrThrow({ where: { id: params.userId } });
    await stripe.billing.meterEvents.create({
      event_name: params.type,
      payload: {
        value: String(params.quantity ?? 1),
        stripe_customer_id: user.stripeCustomerId!,
      },
      identifier: event.id, // Idempotency — same event won't be double-billed
    });

    await db.usageEvent.update({
      where: { id: event.id },
      data: { reportedAt: new Date() },
    });
  } catch (err) {
    console.error('Failed to report usage to Stripe', { eventId: event.id, err });
    // Scheduled job retries unreported events
  }
}
```

## Reconciliation

A scheduled job catches usage events that failed to report to the billing provider:

```typescript
// Runs hourly — reports any unreported usage events
async function reconcileUsageEvents(): Promise<void> {
  const unreported = await db.usageEvent.findMany({
    where: { reportedAt: null, recordedAt: { lte: new Date(Date.now() - 60_000) } },
    take: 500,
  });

  for (const event of unreported) {
    const user = await db.user.findUnique({ where: { id: event.userId } });
    if (!user?.stripeCustomerId) continue;

    await stripe.billing.meterEvents.create({
      event_name: event.type,
      payload: { value: String(event.quantity), stripe_customer_id: user.stripeCustomerId },
      identifier: event.id,
    });

    await db.usageEvent.update({ where: { id: event.id }, data: { reportedAt: new Date() } });
  }
}
```

## Billing Page Requirements

The in-product billing page must display:

```tsx
function UsageBillingPage() {
  const { data } = useCurrentBillingPeriod();

  return (
    <div>
      <h2>Usage & Billing</h2>

      {/* Current period usage */}
      <UsageBar
        label="Emails sent this period"
        used={data.emailsSent}
        limit={data.emailLimit}
      />

      {/* Current period cost estimate */}
      <div>
        <p>Current period cost estimate:</p>
        <p>${(data.basePrice + data.overageCharge).toFixed(2)}</p>
        <p>Base: ${data.basePrice.toFixed(2)} + Overage: ${data.overageCharge.toFixed(2)}</p>
      </div>

      {/* Usage history */}
      <UsageHistoryChart data={data.dailyUsage} />

      {/* Alert threshold management */}
      <UsageAlertSettings
        currentThreshold={data.alertThreshold}
        onUpdate={updateAlertThreshold}
      />
    </div>
  );
}
```

**Required elements:**
- Current period usage and plan limit with a visual progress bar
- Projected end-of-period usage based on daily average
- Current period cost (base + overage accrued so far)
- Per-day or per-hour usage history for the current period
- Alert threshold control (user can set the % at which they receive a warning)

## Related Documents

- [Monetization usage-based](../monetization/usage-based.md)
- [Subscription lifecycle](subscription-lifecycle.md)
- [Stripe integration](../../integrations/providers/stripe/overview.md)
