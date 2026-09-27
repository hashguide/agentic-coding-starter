---
kind: playbook
domain: architecture
topics: [architecture, patterns, behavioral, state]
authority: guidance
---

# State Pattern

The State pattern models an object's lifecycle as a set of explicit, named states and a defined set of valid transitions between them. Instead of tracking state with ad-hoc boolean flags or unconstrained string fields, a state machine makes every possible state explicit and prevents invalid transitions at the code level.

## Definition

A state machine defines the complete set of states an entity can be in, the events that trigger transitions between states, and the actions taken when a transition occurs. Invalid transitions (e.g., transitioning from `canceled` to `active`) are prevented, not just undocumented.

## Why Ad-Hoc Flags Create Invalid State Bugs

Without a state machine:

```typescript
// Ad-hoc flags — what combinations are valid?
type Subscription = {
  isActive: boolean;
  isCanceled: boolean;
  isTrialing: boolean;
  isPastDue: boolean;
  // Can isActive and isCanceled both be true? What does that mean?
};
```

With four boolean flags, there are 16 possible combinations. Most are invalid and undefined. Code that sets `isActive = true` without checking `isCanceled` can produce an incoherent state that silently corrupts downstream behavior.

## Code Example

Subscription lifecycle states as an explicit state machine:

```typescript
type SubscriptionState =
  | 'trialing'
  | 'active'
  | 'past_due'
  | 'canceled'
  | 'incomplete';

type SubscriptionEvent =
  | 'trial_converted'
  | 'payment_succeeded'
  | 'payment_failed'
  | 'grace_period_expired'
  | 'user_canceled'
  | 'reactivated';

const transitions: Record<SubscriptionState, Partial<Record<SubscriptionEvent, SubscriptionState>>> = {
  trialing: {
    trial_converted: 'active',
    user_canceled: 'canceled',
    payment_failed: 'incomplete',
  },
  active: {
    payment_failed: 'past_due',
    user_canceled: 'canceled',
  },
  past_due: {
    payment_succeeded: 'active',
    grace_period_expired: 'canceled',
    user_canceled: 'canceled',
  },
  canceled: {
    reactivated: 'active',
  },
  incomplete: {
    payment_succeeded: 'trialing',
    grace_period_expired: 'canceled',
  },
};

function transition(
  current: SubscriptionState,
  event: SubscriptionEvent,
): SubscriptionState {
  const next = transitions[current]?.[event];
  if (!next) {
    throw new Error(
      `Invalid transition: cannot apply event '${event}' in state '${current}'`,
    );
  }
  return next;
}

// Usage: in the webhook handler
async function handleSubscriptionWebhook(event: StripeWebhookEvent): Promise<void> {
  const subscription = await db.subscription.findUniqueOrThrow({
    where: { stripeSubscriptionId: event.subscriptionId },
  });

  const domainEvent = mapStripeToDomainEvent(event.type);
  const nextState = transition(subscription.state, domainEvent);

  await db.subscription.update({
    where: { id: subscription.id },
    data: { state: nextState, updatedAt: new Date() },
  });
}
```

An attempt to apply an undefined event in the current state throws immediately — the error is proximate and specific, not a downstream corruption that surfaces hours later.

## Why This Matters for Subscription Lifecycle

The subscription state machine maps directly to Stripe's webhook events. See [payments/subscription-lifecycle.md](../../../business/payments/subscription-lifecycle.md) for the complete webhook-to-state-transition mapping.

## Related Documents

- [Payments subscription lifecycle](../../../business/payments/subscription-lifecycle.md)
- [Behavioral patterns overview](overview.md)
- [Observer pattern](observer.md)
