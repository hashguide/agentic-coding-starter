---
kind: playbook
domain: compliance
topics: [compliance, pci, dss]
authority: guidance
---

# PCI DSS

The Payment Card Industry Data Security Standard (PCI DSS) applies to any organization that stores, processes, or transmits cardholder data. Full PCI compliance is a significant operational undertaking — but it can be largely avoided through proper architecture.

## Scope Reduction via Client-Side Tokenization

The key insight: if raw card data never reaches your server, your server is not in PCI scope.

Stripe.js and Stripe Elements handle card entry entirely within Stripe's iframe. The card number, expiry, and CVC are entered directly into a Stripe-hosted form element, processed by Stripe's servers, and returned to your code as a payment method token (e.g., `pm_xxxxxxxx`). This token has no payment value on its own and is safe to pass through your server.

```typescript
// Client-side: collect card data via Stripe Elements, never touch raw numbers
import { loadStripe } from '@stripe/stripe-js';
import { Elements, useStripe, useElements, CardElement } from '@stripe/react-stripe-js';

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!);

function CheckoutForm() {
  const stripe = useStripe();
  const elements = useElements();

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    if (!stripe || !elements) return;

    // Stripe handles card capture; we receive only a token
    const { error, paymentMethod } = await stripe.createPaymentMethod({
      type: 'card',
      card: elements.getElement(CardElement)!,
    });

    if (error) {
      // Display error to user — never log card data
      setError(error.message ?? 'Payment failed');
      return;
    }

    // Send only the token to your server — no card data
    await fetch('/api/subscriptions', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ paymentMethodId: paymentMethod.id }),
    });
  }
}
```

```typescript
// Server-side: attach the payment method to a Stripe customer, create subscription
// Your server never sees card numbers — only opaque payment method IDs
app.post('/api/subscriptions', requireAuth, asyncHandler(async (req, res) => {
  const { paymentMethodId } = z.object({ paymentMethodId: z.string() }).parse(req.body);

  const user = await db.user.findUniqueOrThrow({ where: { id: req.userId } });

  await stripe.paymentMethods.attach(paymentMethodId, { customer: user.stripeCustomerId });

  const subscription = await stripe.subscriptions.create({
    customer: user.stripeCustomerId,
    items: [{ price: process.env.STRIPE_PRICE_ID }],
    default_payment_method: paymentMethodId,
  });

  await db.user.update({
    where: { id: req.userId },
    data: { stripeSubscriptionId: subscription.id },
  });

  res.json({ subscriptionId: subscription.id });
}));
```

## PCI SAQ A

With Stripe.js/Elements handling all card capture, your PCI scope is limited to SAQ A (Self-Assessment Questionnaire A) — the minimal tier. SAQ A requires:

- HTTPS on all payment pages
- No card data stored on your systems
- Annual self-assessment questionnaire completion
- Quarterly external vulnerability scans (often waived at SAQ A level with certain processors)

You do not need annual on-site audits, penetration testing at the PCI level, or the full 12-requirement PCI DSS program.

## Security

**Any server that touches raw card numbers is in full PCI scope.** Full scope means annual audits by a Qualified Security Assessor (QSA), all 12 PCI DSS requirement domains, penetration testing, and ongoing compliance maintenance costing tens of thousands of dollars annually.

Never build a custom card input form that posts to your server. Never log request bodies containing card numbers. Never receive card numbers from your client or any third party and forward them to Stripe — always let the Stripe client library tokenize at the point of capture.

## Related Documents

- [Payments provider options](../business/payments/provider-options.md)
- [Stripe integration](../integrations/providers/stripe/overview.md)
- [Security transport](../security/transport.md)
- [Compliance overview](overview.md)
