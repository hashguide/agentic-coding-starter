---
kind: playbook
domain: business
topics: [business, monetization, freemium]
authority: guidance
---

# Freemium

Freemium gives users access to a meaningful subset of the product for free, with a path to paid tiers for more features, volume, or capability. Done well, it reduces signup friction and creates a large pool of potential upgraders. Done poorly, it creates a large pool of users who never upgrade and a cost center that drains the business.

## What Should Be Free

The free tier must include enough of the product for users to experience the core value. A free tier that doesn't let users accomplish anything meaningful produces no upgrades — users leave before they understand why they'd pay.

**Rules for what stays free:**
- The one or two features that demonstrate the core value proposition most clearly
- Enough volume to actually use those features (e.g., 3 projects, 100 emails/month — not 1 project or 10 emails)
- Sufficient depth to evaluate whether the product fits the user's workflow

**Rules for what goes behind the paywall:**
- Features used primarily by power users or teams
- Volume that significantly increases infrastructure cost per user
- Features that create team-level or enterprise-level value (SSO, audit logs, admin controls)
- Integrations that are valuable but not core to the solo-user value prop

## Where the Paywall Sits

The paywall should sit after the user has experienced value — not before.

**Wrong:** Hitting a paywall before you can create your first project. The user doesn't know yet whether the product is worth paying for.

**Right:** Creating your first 3 projects, experiencing the value, and then being prompted to upgrade when you try to create the 4th.

The upgrade prompt appears inline, in context, when the user hits the limit — not as a random interstitial. "You've reached the limit for your free plan. Upgrade to Pro to create unlimited projects."

## The Sustainability Rule

The free tier must be sustainable to run at scale. If every free user costs you $5/month in infrastructure and only 3% convert to paid, you need the average paid revenue to be at least $167/user to break even on the free tier. If the math doesn't work, the free tier is a liability.

Reduce the free tier's infrastructure cost per user:
- Enforce storage limits strictly
- Archive or delete data for accounts inactive > 90 days (with notice)
- Limit background jobs and automation for free accounts
- Do not provide dedicated compute for free-tier workloads

## Freemium vs. Free Trial

| Model | Structure | When to use |
|---|---|---|
| **Freemium** | Free forever, limited features | High natural virality; simple product; viral acquisition matters |
| **Free trial** | Full access, time-limited | Complex product where full access is needed to evaluate; B2B sales |

Many products use both: a freemium tier for low-complexity use cases and a free trial of a higher tier for users evaluating the full product for team or enterprise use.

## Related Documents

- [Monetization overview](overview.md)
- [Trials](trials.md)
- [Upgrade flows](upgrade-flows.md)
- [Pricing strategy](pricing-strategy.md)
