---
kind: playbook
domain: business
topics: [business, monetization, usage, based]
authority: guidance
---

# Usage-Based Billing

Usage-based billing (also called metered billing or pay-as-you-go) charges customers based on how much of the product they consume, not a flat fee. It aligns cost with value: users who get more value pay more.

## Choosing a Meterable Unit

The unit must:
1. Be closely correlated with the value the user gets from the product
2. Be easy for the user to understand and predict
3. Be measurable in your system before billing

| Product type | Good metered unit | Why |
|---|---|---|
| AI writing tool | Tokens generated / words written | Directly reflects compute cost + value delivered |
| Email platform | Emails sent | Direct relationship between usage and value |
| Storage product | GB stored | Intuitive and correlates with infrastructure cost |
| API | API requests | Developer-native mental model |
| SMS platform | Messages sent | Direct cost relationship |

**Bad metered units:** "complexity points", "workflow runs" (vague), "active users" (for a usage product, not a seat product).

## Spending Cap and Usage Alert Pattern

Unpredictable bills are the #1 source of churn in usage-based products. Users who receive a surprise $300 charge cancel and post about it. The product must proactively inform users of their usage trajectory.

**Required pattern:**
1. **At 80% of plan limit:** Email alert + in-product notification: "You've used 80% of your monthly [units]. At this rate, you'll hit your limit in X days."
2. **At 100% of plan limit:** 
   - Option A (hard stop): Service stops or degrades for the billing period. User is prompted to upgrade.
   - Option B (overage billing): Service continues; overage is billed at a defined per-unit rate. User must opt into overage billing explicitly.

**Never silently charge overages.** A user who signed up for a $29/mo plan and receives a $147 bill because of unannounced overage pricing will churn and leave a negative review.

## In-Product Usage Dashboard

Users must be able to see their current usage and projected bill within the product at any time. This is not optional — it is the mechanism that allows users to stay within budget.

**Required elements:**
```
Billing period: June 1 – June 30
Usage this period: 8,200 / 10,000 emails (82%)
Projected usage by end of period: ~9,400 emails
Current period cost: $29.00 (base) + $0.00 (overage)

[Upgrade to increase your limit]  [View usage history]
```

Display:
- Current period usage and limit, with a visual progress bar
- Projected end-of-period usage (based on daily average)
- Current period cost (base + any overage accrued)
- Usage history (per-day or per-hour breakdown, last 30 days)
- Clear path to upgrade if approaching the limit

## Metering in Your Own Database

Meter usage in your own database first; reconcile with the billing provider second. Do not rely solely on the billing provider's metering records.

```typescript
// Record each usage event in your own database
async function recordUsage(params: {
  userId: string;
  type: 'email_sent' | 'api_request' | 'token_generated';
  quantity: number;
  metadata?: Record<string, unknown>;
}) {
  await db.usageEvent.create({
    data: {
      userId: params.userId,
      type: params.type,
      quantity: params.quantity,
      metadata: params.metadata ?? {},
      recordedAt: new Date(),
    },
  });

  // Report to Stripe metering (for billing reconciliation)
  await stripe.billing.meterEvents.create({
    event_name: params.type,
    payload: {
      value: String(params.quantity),
      stripe_customer_id: await getStripeCustomerId(params.userId),
    },
  });
}
```

Your database records are the source of truth for disputes. Stripe's records are the source of truth for billing.

## Related Documents

- [Monetization overview](overview.md)
- [Payments usage-based](../payments/usage-based.md)
- [Upgrade flows](upgrade-flows.md)
- [Stripe integration](../../integrations/providers/stripe/overview.md)
