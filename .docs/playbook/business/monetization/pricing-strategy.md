---
kind: playbook
domain: business
topics: [business, monetization, pricing, strategy]
authority: guidance
---

# Pricing Strategy

Pricing is a design decision with psychological dimensions. The numbers you choose, how many tiers you create, and what you label the "recommended" plan all influence purchasing behavior — often more than the absolute price level.

## Anchoring

The presence of a higher-priced option makes the middle option feel more reasonable. Without a $99/mo Enterprise tier, $29/mo feels expensive. With it, $29/mo feels moderate.

**The anchor effect:** Users who see three tiers ($9 / $29 / $99) typically choose the middle tier at higher rates than users who see only ($9 / $29). The $99 tier justifies the $29 choice even for users who would never buy at $99.

**Practical implementation:**
- Always include a higher-priced tier, even if you expect few buyers at that price
- The highest tier can be "Contact us" for custom pricing — it anchors the perception of value without requiring a self-serve price
- Visually highlight the middle tier as "Most popular" or "Recommended" to reduce decision fatigue

## Value Metrics

A value metric is the unit your pricing scales with. It should scale proportionally with the value the customer gets from the product.

| Product type | Good value metric | Weak value metric |
|---|---|---|
| Email marketing tool | Subscribers / emails sent | Users (teams typically have 1-2 accounts) |
| Project management | Team members (seats) | Projects (power users create many projects) |
| API product | API calls / requests | Users |
| Analytics tool | Events tracked | Dashboards (usage doesn't correlate with dashboards) |

A weak value metric misaligns price with value. A large customer who sends 100,000 emails paying the same as one who sends 1,000 means you're leaving money on the table while also pricing out the small user.

## MVP Pricing Rule: Keep It Simple

Two or three tiers is sufficient for an MVP:
- **Free / Starter:** Core value, limited volume or features — enough to validate the product
- **Pro:** Full feature access + meaningful volume — the target purchase for most users
- **Enterprise (optional):** Custom pricing, SLAs, SSO, dedicated support

More than three tiers at MVP stage creates analysis paralysis without providing revenue intelligence you can't get from simpler pricing.

**Raise prices later.** Pricing too low at MVP is a recoverable mistake. Pricing structure that forces a confusing conversation with every prospective customer is harder to fix. Start simple; add complexity only when your data supports it.

## Annual Billing Discount

Offering a 15–20% discount for annual billing improves cash flow (you receive 12 months upfront) and reduces churn (customers who pay annually have lower churn rates). Most SaaS products offer this from day one.

Show annual pricing with the monthly equivalent prominently: "$24/mo, billed annually as $288."

## Related Documents

- [Monetization overview](overview.md)
- [Pricing layout](../../experience/layouts/pricing.md)
- [Freemium design](freemium.md)
- [Upgrade flows](upgrade-flows.md)
