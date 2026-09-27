---
kind: playbook
domain: operations
topics: [operations, metrics]
authority: guidance
---

# Product Metrics

Define and instrument these metrics before launch, not retroactively. A metric you cannot calculate for the first week of launch is a metric you cannot trend.

## Core Metrics — Definitions

### Activation Rate
**Definition:** Percentage of new signups who complete the "aha moment" action within 7 days.

The aha moment is the first action that correlates with long-term retention — not account creation, but the first meaningful use of your product's core value. Examples: "created first project," "connected first integration," "invited first teammate."

```
Activation Rate = (Users who completed aha moment) / (New signups in same cohort) × 100
```

Instrument: log the specific action as an event and join against the signup cohort.

### Trial-to-Paid Conversion Rate
**Definition:** Percentage of trial users who convert to a paid plan before trial expiry.

```
Trial Conversion Rate = (Users who paid) / (Users who started trial) × 100
```

Segment by source, onboarding completion, and activation status to find the highest-signal predictor of conversion.

### MRR (Monthly Recurring Revenue)
**Definition:** Total monthly subscription revenue, excluding one-time charges and non-recurring payments.

```
MRR = Sum of all active monthly subscriptions (monthly price)
    + Sum of all active annual subscriptions (annual price / 12)
```

Track: New MRR, Expansion MRR (upgrades), Contraction MRR (downgrades), Churned MRR, Net New MRR.

### Churn Rate
**Definition:** Percentage of customers who cancel in a given month.

```
Monthly Churn Rate = Churned customers in month / Customers at start of month × 100
```

Revenue churn (MRR churned / MRR at start) is more informative than customer churn when plans are priced differently.

### NPS (Net Promoter Score)
**Definition:** Score from -100 to +100 measuring customer loyalty based on the question "How likely are you to recommend [product] to a friend or colleague?" (0–10 scale).

```
NPS = % Promoters (9-10) - % Detractors (0-6)
```

Survey timing: 30 days after activation, then every 90 days. Survey the activated users, not all signups.

### Time-to-First-Value (TTFV)
**Definition:** Median time from signup to completion of the aha moment.

```
TTFV = Median(aha_moment_timestamp - signup_timestamp)
       for users who activated in the period
```

Lower is better. A TTFV over 24 hours usually indicates friction in onboarding. Target: under 60 minutes for most SaaS products.

## Instrumentation

```typescript
// Emit events at each key milestone
// lib/analytics.ts

export async function trackActivation(userId: string) {
  await db.analyticsEvent.create({
    data: { userId, event: 'activated', timestamp: new Date() },
  });
  // Also send to your analytics provider (PostHog, Mixpanel, etc.)
  posthog.capture({ distinctId: userId, event: 'user_activated' });
}

export async function trackTrialConverted(userId: string, planId: string) {
  await db.analyticsEvent.create({
    data: { userId, event: 'trial_converted', metadata: { planId }, timestamp: new Date() },
  });
}
```

## When to Check These Metrics

| Metric | Review cadence | Intervene when |
|---|---|---|
| Activation rate | Weekly | Drops below 30% in a cohort |
| Trial conversion | Weekly | Drops below historical baseline |
| MRR | Daily | Any significant unexpected drop |
| Churn | Monthly | Exceeds 5% monthly customer churn |
| NPS | Quarterly | Score below +20 |
| TTFV | Monthly | Increases significantly without a product change explanation |

## Related Documents

- [Ops feedback](feedback.md)
- [Design ux-flows new user](../experience/journeys/new-user.md)
- [Monetization overview](../business/monetization/overview.md)
