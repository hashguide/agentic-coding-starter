---
kind: playbook
domain: architecture
topics: [architecture, patterns, creational, factory]
authority: guidance
---

# Factory Pattern

The Factory pattern creates objects without requiring the caller to specify the exact class or implementation. Instead, the caller provides a configuration value (a type string, an enum, an environment variable) and the factory returns the appropriate object. The caller works with an interface; the factory owns the mapping from configuration to concrete implementation.

## Definition

A factory is a function or class whose responsibility is to instantiate and return the correct implementation of an interface, based on runtime context.

## Code Example

Creating payment provider clients based on configuration:

```typescript
interface PaymentProvider {
  createCheckoutSession(params: CheckoutParams): Promise<CheckoutSession>;
  cancelSubscription(subscriptionId: string): Promise<void>;
}

class StripeProvider implements PaymentProvider {
  async createCheckoutSession(params: CheckoutParams): Promise<CheckoutSession> {
    // Stripe-specific implementation
  }
  async cancelSubscription(subscriptionId: string): Promise<void> {
    // Stripe-specific implementation
  }
}

class PaddleProvider implements PaymentProvider {
  async createCheckoutSession(params: CheckoutParams): Promise<CheckoutSession> {
    // Paddle-specific implementation
  }
  async cancelSubscription(subscriptionId: string): Promise<void> {
    // Paddle-specific implementation
  }
}

function createPaymentProvider(): PaymentProvider {
  const provider = process.env.PAYMENT_PROVIDER;
  switch (provider) {
    case 'stripe':
      return new StripeProvider();
    case 'paddle':
      return new PaddleProvider();
    default:
      throw new Error(`Unknown payment provider: ${provider}`);
  }
}

// Callers work with the interface, never the concrete class
const payment = createPaymentProvider();
await payment.createCheckoutSession(params);
```

## When to Use This

- You need to swap implementations based on environment or configuration
- You want to test the system with a mock implementation without changing callers
- You're adapting multiple third-party providers to a single internal interface (see [Strategy pattern](../behavioral/strategy.md) for the behavioral equivalent)
- Object creation is complex enough that you don't want that logic scattered across callers

## When This Is Overkill

- There is only one implementation, and no realistic need for a second
- The "configuration" is a boolean (`if (isProd)`) — an `if` statement is clearer than a factory
- The objects being created are simple value types with no behavior

The Abstract Factory variant (a factory that creates families of related objects) is worth reaching for when you have multiple related types that always need to change together. For most SaaS products, a simple factory function is sufficient.

## Related Documents

- [Strategy pattern](../behavioral/strategy.md)
- [Adapter pattern](../structural/adapter.md)
- [Patterns overview](../overview.md)
