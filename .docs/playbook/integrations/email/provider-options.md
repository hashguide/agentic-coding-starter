---
kind: playbook
domain: integrations
topics: [integrations, email, provider, options]
authority: guidance
---

# Email Provider Options

Transactional email providers differ in deliverability track record, pricing, developer experience, and template tooling. The right choice depends on volume, technical preferences, and budget.

## Comparison Table

| Dimension | Resend | Postmark | SendGrid |
|---|---|---|---|
| **Deliverability reputation** | Excellent (modern, clean IP pools) | Excellent (dedicated transactional focus, long track record) | Good (large shared pools; deliverability varies more by sending reputation) |
| **Pricing model** | Generous free tier (3,000 emails/mo); then pay-per-volume | Pay-per-volume (no free tier beyond trial); transparent overage | Free tier (100/day); then subscription tiers |
| **Developer experience** | Excellent — React Email + REST API; first-class TypeScript SDK | Very good — REST API, webhooks, clean docs | Good — REST API + SMTP; more complex dashboard |
| **Template tooling** | React Email (JSX components) + WYSIWYG | Visual template editor + Postmark template language | Dynamic Templates (Handlebars) + Inbound Parse |
| **Email types** | Transactional only | Transactional (very strong) | Transactional + marketing (can be a negative — shared reputation pool) |
| **Webhook events** | Delivered, bounced, complained | Delivered, bounced, spam, unsubscribed | Delivered, opened, clicked, bounced, spam, unsubscribed |
| **Inbound email parsing** | No | Basic | Yes (Inbound Parse) |

## Recommendation for a Solo Builder

**Resend** is one option for projects prioritizing a simple developer experience:
- Generous free tier covers early-stage volumes with no credit card required
- React Email integration allows you to build emails as React components (no custom templating language to learn)
- Modern API with first-class TypeScript SDK
- Clean deliverability reputation as a newer, dedicated transactional provider

**Postmark** is the recommended alternative if deliverability is the primary concern. It has a 15+ year track record as a transactional-only provider and is often cited as having the best deliverability in the industry. No free tier, but the deliverability investment is worth it for products where email reliability is critical.

**SendGrid** is appropriate when you need its specific features: inbound email processing, bulk marketing emails alongside transactional, or if it's already the company standard. Be aware that sharing IP pools with many senders (including spammers) means your deliverability depends on your own sending reputation more than the pool's.

## Related Documents

- [Transactional emails](transactional.md)
- [Email notifications](notifications.md)
- [SendGrid integration](../providers/sendgrid/overview.md)
