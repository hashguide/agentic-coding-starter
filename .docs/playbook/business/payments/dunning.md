---
kind: playbook
domain: business
topics: [business, payments, dunning]
authority: guidance
---

# Dunning

Dunning is the process of recovering failed payments. A payment failure is not a cancellation — the user did not choose to stop paying; their card failed. A structured dunning sequence recovers 20–40% of failed payments before they become involuntary churn.

## Failed Payment Recovery Strategy

### Smart Retries (Stripe Handles This)

Stripe automatically retries failed payments on a schedule (typically days 5, 10, and 15 after failure by default). Enable Smart Retries in the Stripe dashboard — it uses machine learning to time retries when success probability is highest (e.g., after the user's payday, when the card network signals the account is in good standing).

Do not retry more aggressively than your payment processor allows — excessive retries can result in card blocks.

### Dunning Email Sequence

The dunning emails run in parallel with Stripe's retries:

| Day | Email | Goal |
|---|---|---|
| Day 0 (immediately on failure) | "Your payment failed" | Inform the user; provide a direct update payment method link |
| Day 3 | "Reminder: update your payment method" | Catch users who missed the first email |
| Day 7 | "Final notice: access ends soon" | Create urgency; explain what will happen if payment isn't resolved |
| Day N (grace period end) | "Your access has been downgraded" | Confirm the downgrade; provide a path to restore access |

**Critical requirement:** Every dunning email must include a one-click link to update the payment method — not a link to log in and then navigate to billing settings. The fewer steps, the higher the recovery rate.

With Stripe Billing Portal:

```typescript
async function generateBillingPortalUrl(userId: string): Promise<string> {
  const user = await db.user.findUniqueOrThrow({ where: { id: userId } });
  const session = await stripe.billingPortal.sessions.create({
    customer: user.stripeCustomerId!,
    return_url: `${process.env.APP_URL}/dashboard`,
  });
  return session.url;
}
```

Include this URL directly in the dunning email — the user clicks it, lands in the Stripe billing portal, updates their card, and is done.

## Grace Period

A grace period is the window between the first payment failure and the actual downgrade. The user retains access during this window.

**Recommended grace period:** 7 days (enough to allow card updates across a billing cycle; short enough to minimize revenue leakage from unresponsive accounts).

During the grace period:
- Access is maintained
- Dunning emails are sent per the sequence above
- The in-app banner shows "Your payment failed. Update your payment method to avoid losing access."

After the grace period:
- Downgrade to the free tier (if one exists) or suspend access
- Send the downgrade confirmation email

## Stripe Smart Retry Configuration

```typescript
// Set at the Stripe level — not in application code
// Stripe dashboard → Billing → Settings → Subscription and emails
// Enable "Smart Retries" and configure the retry schedule

// Application handling after retries are exhausted:
// customer.subscription.deleted or subscription.status = 'canceled'
// → downgrade user in your database
```

## Related Documents

- [Subscription lifecycle](subscription-lifecycle.md)
- [Payments provider options](provider-options.md)
- [Email transactional](../../integrations/email/transactional.md)
- [Stripe integration](../../integrations/providers/stripe/overview.md)
