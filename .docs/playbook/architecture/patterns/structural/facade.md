---
kind: playbook
domain: architecture
topics: [architecture, patterns, structural, facade]
authority: guidance
---

# Facade Pattern

The Facade pattern provides a simplified interface over a complex subsystem. Callers interact with the facade and never need to understand the internal complexity. The facade coordinates the underlying components and presents a single, coherent API.

## Definition

A facade is a class or module that exposes a simplified interface over a set of underlying subsystems that have their own complex interfaces, initialization requirements, or state.

## Code Example

A `BillingService` facade hiding multiple Stripe API calls:

```typescript
// Without a facade, callers must orchestrate multiple Stripe calls
// and handle the interactions between them.

// With a facade:
class BillingService {
  private stripe: Stripe;

  constructor() {
    this.stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
      apiVersion: '2024-11-20.acacia',
    });
  }

  async startSubscription(params: {
    userId: string;
    email: string;
    priceId: string;
    successUrl: string;
    cancelUrl: string;
  }): Promise<{ checkoutUrl: string }> {
    // Create customer if needed (Stripe call 1)
    let customer = await this.findOrCreateCustomer(params.userId, params.email);

    // Create checkout session (Stripe call 2)
    const session = await this.stripe.checkout.sessions.create({
      customer: customer.id,
      mode: 'subscription',
      line_items: [{ price: params.priceId, quantity: 1 }],
      success_url: params.successUrl,
      cancel_url: params.cancelUrl,
    });

    return { checkoutUrl: session.url! };
  }

  async cancelSubscription(userId: string): Promise<void> {
    const subscription = await this.getUserSubscription(userId);
    if (!subscription) return;

    // Cancel at period end (Stripe call 1)
    await this.stripe.subscriptions.update(subscription.stripeSubscriptionId, {
      cancel_at_period_end: true,
    });

    // Record the cancellation in our DB (internal call)
    await db.subscription.update({
      where: { userId },
      data: { canceledAt: new Date() },
    });
  }

  async issueRefund(invoiceId: string, amountCents: number): Promise<void> {
    const invoice = await db.stripeInvoice.findUnique({ where: { id: invoiceId } });
    if (!invoice?.paymentIntentId) throw new NotFoundError('INVOICE_NOT_FOUND');

    await this.stripe.refunds.create({
      payment_intent: invoice.paymentIntentId,
      amount: amountCents,
    });
  }

  private async findOrCreateCustomer(userId: string, email: string): Promise<Stripe.Customer> {
    const existing = await db.user.findUnique({
      where: { id: userId },
      select: { stripeCustomerId: true },
    });

    if (existing?.stripeCustomerId) {
      return this.stripe.customers.retrieve(existing.stripeCustomerId) as Promise<Stripe.Customer>;
    }

    const customer = await this.stripe.customers.create({ email, metadata: { userId } });
    await db.user.update({ where: { id: userId }, data: { stripeCustomerId: customer.id } });
    return customer;
  }

  private async getUserSubscription(userId: string) {
    return db.subscription.findUnique({ where: { userId } });
  }
}

// Callers use three clean methods — no Stripe SDK knowledge required
const billing = new BillingService();
const { checkoutUrl } = await billing.startSubscription({ userId, email, priceId, successUrl, cancelUrl });
```

## When a Facade Is Worth the Extra Layer

- The subsystem has multiple steps that must be coordinated in a specific order
- Callers need only a small subset of the subsystem's full capability
- You want to insulate the rest of the codebase from changes in the subsystem's API
- The subsystem has an unfamiliar or complex SDK that you want to normalize

## When to Skip the Facade

If you're making a single API call with a simple response and there's no coordination involved, a facade adds indirection without benefit. Use the SDK directly and introduce the facade when coordination becomes complex enough to warrant it.

## Related Documents

- [Adapter pattern](adapter.md)
- [Proxy pattern](proxy.md)
- [Stripe integration overview](../../../integrations/providers/stripe/overview.md)
