---
kind: playbook
domain: integrations
topics: [integrations, providers, stripe]
authority: guidance
---

# Stripe Setup

Stripe is the payment processor for subscriptions, one-time charges, and (via Connect) marketplace payments. This document covers initial account setup, API key configuration, and webhook signing.

## Account Creation

1. Go to [stripe.com](https://stripe.com) and create an account
2. Complete identity verification (required for live mode payouts)
3. Set the account name to match the product (visible on customer receipts and statements)
4. Configure the Stripe Customer Portal (Billing → Customer Portal) — this handles self-service subscription management and reduces support overhead

## API Keys

Stripe provides two key pairs: **test mode** and **live mode**. Each pair contains a publishable key (safe to expose in client-side code) and a secret key (server-only).

Locate them in: **Developers → API keys**

```
Test publishable:  pk_test_...
Test secret:       sk_test_...
Live publishable:  pk_live_...
Live secret:       sk_live_...
```

## Restricted API Keys

Instead of using the full secret key, create restricted keys scoped to the minimum permissions required. A key used only for subscription billing should not have permission to modify products or access Connect accounts.

**Developers → API keys → Create restricted key**

Typical permissions for a subscription billing key:
- Customers: Read and write
- Subscriptions: Read and write
- Payment intents: Read and write
- Payment methods: Read and write
- Invoices: Read
- Billing portal: Read and write

## Webhook Configuration

**Developers → Webhooks → Add endpoint**

Configure the endpoint URL (`https://yourdomain.com/webhooks/stripe`) and select the events to listen for. At minimum:
- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `invoice.payment_succeeded`
- `invoice.payment_failed`
- `customer.subscription.trial_will_end`

After creating the endpoint, click "Reveal" to see the **Webhook signing secret** (`whsec_...`). Add this to your environment variables.

```typescript
// environment variables required
STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

## Environment Configuration

```typescript
import Stripe from 'stripe';

export const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-04-10',
  typescript: true,
});
```

## Security

**Use restricted keys scoped to minimum permissions.** A leaked secret key with full permissions gives an attacker access to all Stripe capabilities on your account — refunds, transfers, Connect account modifications. A restricted key with subscription-only permissions limits the blast radius.

**Maintain completely separate key sets per environment.** Test and live keys must never be mixed. A test webhook secret used in production will reject all real webhooks. A live key used in development risks creating real charges during testing.

**Never commit keys to git.** Even momentarily. A key committed and immediately removed is still exposed — git history is permanent and forks may have already cloned it.

## Related Documents

- [Stripe subscriptions](subscriptions.md)
- [Stripe webhooks](webhooks.md)
- [Stripe testing](testing.md)
- [Payments subscription lifecycle](../../../business/payments/subscription-lifecycle.md)
