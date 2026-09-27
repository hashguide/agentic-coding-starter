---
kind: playbook
domain: business
topics: [business, payments, provider, options]
authority: guidance
---

# Payments Provider Options

Stripe, Paddle, and Lemon Squeezy are common self-serve payment platforms for software, digital products, and online services. The primary decision axis is merchant of record vs. payment processor — which determines who is legally the seller and who handles tax compliance.

## Comparison Table

| Dimension | Stripe | Paddle | Lemon Squeezy |
|---|---|---|---|
| **Merchant of record** | No — you are the merchant | Yes — Paddle is the merchant | Yes — Lemon Squeezy is the merchant |
| **Tax handling** | You handle (need tax software: TaxJar, Avalara) | Handles all global VAT/GST/sales tax automatically | Handles all global VAT/GST/sales tax automatically |
| **Fee structure** | 2.9% + $0.30 per transaction | 5–10% revenue share (includes all fees) | 5% + $0.50 per transaction (blended) |
| **Supported models** | Subscriptions, one-time, usage-based, complex billing | Subscriptions, one-time, usage-based | Subscriptions, one-time, digital products |
| **Developer experience** | Excellent — industry-leading API, SDK, docs | Good — REST API, webhooks | Good — REST API, webhooks |
| **Dashboard / admin** | Comprehensive | Good | Good, simpler |
| **Payout speed** | 2 business days | Weekly | Weekly |
| **Fraud tooling** | Stripe Radar (excellent) | Basic | Basic |

## When Merchant-of-Record (Paddle / Lemon Squeezy) is Preferable

For a solo international seller, being your own merchant of record means:
- Registering for VAT/GST in every country where you exceed local thresholds
- Filing quarterly or annual tax returns in multiple jurisdictions
- Understanding the requirements for digital goods taxation in the EU (OSS scheme), UK, Australia, etc.

This is a non-trivial ongoing compliance burden. A merchant-of-record provider (Paddle, Lemon Squeezy) becomes the seller of record on every transaction — they handle all tax obligations, remit tax to the relevant authorities, and indemnify you against tax compliance failures.

**Choose Paddle or Lemon Squeezy if:**
- You are a solo builder or small team without a dedicated finance function
- You sell to international customers and don't want to manage global tax compliance
- Simplicity of operations is valued over fee optimization

**Choose Stripe if:**
- You are in a geography where tax compliance is simple (e.g., US-only sales early on)
- You need Stripe's advanced features (Stripe Connect, complex billing, Stripe Tax add-on, Stripe Radar)
- You have the infrastructure to handle tax separately (TaxJar, Avalara, or you hire it out)
- Enterprise customers require Stripe as a payment processor for their vendor approval process

## Recommendation

**Default for solo international builders:** Lemon Squeezy (simpler, lower minimum complexity) or Paddle (better tooling for larger volumes).

**Default for US-focused products with technical complexity:** Stripe.

**Products with marketplace/Connect requirements:** Stripe (Stripe Connect has no equivalent in Paddle/LemonSqueezy).

## Related Documents

- [Stripe integration](../../integrations/providers/stripe/overview.md)
- [PCI DSS scope reduction](../../compliance/pci-dss.md)
- [Subscription lifecycle](subscription-lifecycle.md)
- [One-time payments](one-time-payments.md)
- [Monetization overview](../monetization/overview.md)
