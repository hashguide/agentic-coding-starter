---
kind: playbook
domain: architecture
topics: [architecture, patterns, behavioral, observer]
authority: guidance
---

# Observer Pattern

The Observer pattern (also called publish/subscribe or event emitter) enables decoupled event-driven systems. When something notable happens — a user is created, a payment succeeds, an invoice is sent — the event is published to a central event bus. Any number of subscribers react independently, without the publisher knowing who is listening.

## Definition

An observable (publisher) emits named events when its state changes. Observers (subscribers/listeners) register interest in specific events and are called when those events fire. The publisher never calls observers directly — it emits, and the event system handles delivery.

## Code Example

Emitting a `user.created` event that triggers multiple independent listeners:

```typescript
import EventEmitter from 'events';

// Typed event emitter
interface AppEvents {
  'user.created': [{ userId: string; email: string; plan: string }];
  'invoice.sent': [{ userId: string; invoiceId: string; clientEmail: string; total: number }];
  'payment.succeeded': [{ userId: string; amount: number; invoiceId: string }];
}

class AppEventBus extends EventEmitter {
  emit<K extends keyof AppEvents>(event: K, ...args: AppEvents[K]): boolean {
    return super.emit(event, ...args);
  }

  on<K extends keyof AppEvents>(event: K, listener: (...args: AppEvents[K]) => void): this {
    return super.on(event, listener);
  }
}

export const eventBus = new AppEventBus();

// Publisher — the user service knows nothing about welcome emails or analytics
async function createUser(email: string, password: string): Promise<User> {
  const user = await db.user.create({ data: { email, passwordHash: hash(password) } });
  eventBus.emit('user.created', { userId: user.id, email, plan: 'free' });
  return user;
}

// Subscriber 1: welcome email
eventBus.on('user.created', async ({ userId, email }) => {
  await emailService.sendTransactional({
    to: email,
    subject: 'Welcome!',
    html: welcomeEmailTemplate({ userId }),
    text: 'Welcome to the app.',
  });
});

// Subscriber 2: analytics
eventBus.on('user.created', ({ userId, plan }) => {
  analytics.track('user_signed_up', { userId, plan });
});

// Subscriber 3: onboarding background job
eventBus.on('user.created', async ({ userId }) => {
  await jobQueue.add('onboarding', { userId });
});
```

Adding a fourth reaction (e.g., a Slack notification) is a new subscriber registration — no change to the publisher.

## When Event-Driven Decoupling Is Worth It

- Multiple components need to react to the same occurrence
- The publisher should not be aware of its consumers
- Reactions are independent and can fail without affecting each other
- You want to add/remove reactions without modifying the code that triggers the event

## When Direct Calls Are Simpler

If there is exactly one reaction to an event, emit-and-subscribe adds indirection without benefit. A direct function call is clearer. Use the observer pattern when you have two or more independent reactions, or when you anticipate needing to add more.

For reactions that must be reliable (emails, background jobs), consider routing through a persistent job queue instead of an in-process event emitter — in-process emitters lose events if the process crashes.

## Related Documents

- [Workers overview](../../../backend/async/overview.md)
- [Strategy pattern](strategy.md)
- [Behavioral patterns overview](overview.md)
