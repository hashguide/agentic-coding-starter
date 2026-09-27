---
kind: playbook
domain: operations
topics: [operations, feedback]
authority: guidance
---

# Feedback Collection

User feedback is the highest-signal input for product decisions. This document covers collection channels and how to use support volume as a product direction signal.

## Feedback Collection Channels

### In-App Feedback Widget

Install a persistent feedback widget (a button in the corner of every page) that allows users to submit feedback or report bugs without leaving the app. Options:

- **Canny** — captures feedback, allows voting, integrates with roadmap
- **Usersnap** — screenshot-based feedback, good for bugs
- **Sentry User Feedback** — built into Sentry, tied to sessions and errors
- **Custom form** — a textarea that posts to your support email or database

The widget must be opt-in (not an unsolicited prompt) and must not interrupt the user's workflow. A small floating button is sufficient.

### NPS Survey

Send an NPS survey to activated users:
- **Timing:** 30 days after the user completes the activation action (not 30 days after signup — many users haven't activated by then)
- **Channel:** Email (higher response rate than in-app modals)
- **Follow-up:** For detractors (0–6), send a one-line reply: "Thanks for your honesty. What's the biggest thing we could improve?" Read every response.

Do not automate follow-up responses to detractors. A human reply within 48 hours converts more detractors than a well-crafted automated sequence.

### User Interviews

Schedule 30-minute video calls with:
- 2–3 power users (understand what they love)
- 2–3 recently churned users (understand what drove them away)
- 2–3 users who activated but never converted to paid

Use a consistent question set:
1. "Walk me through how you use [product] today."
2. "What was the last thing that frustrated you?"
3. "What would make you recommend this to a colleague?"
4. "What's missing for you?"

Do not pitch or defend during the interview. Listen.

## Support Volume as a Product Signal

**Every recurring support category is a documentation gap or a UX failure.**

Track support ticket topics weekly:

```
Week 1 topics:
- "How do I export my data?" (8 tickets) → Write the export tutorial
- "Can I invite teammates?" (3 tickets) → Feature doesn't exist or is not discoverable
- "My payment failed, what do I do?" (2 tickets) → Improve dunning email clarity
```

If the same question appears 3+ times in a week, it is a product problem, not a user problem. Do not just answer it — fix the root cause:
- If a feature is confusing: improve the UX
- If a feature is undiscoverable: add a tooltip, empty state CTA, or in-app prompt
- If a feature is missing: add it to the roadmap

## Churned User Outreach

When a user cancels, send a single, personal-feeling email within 24 hours:

```
Subject: Quick question about [Product]

Hi [Name],

I saw you cancelled your [Product] account. I'm not here to change your mind —  
I just want to understand if there's something we could do better.

One question: What was the main reason you decided to stop using [Product]?

[Your name]
```

Keep it short. One question. Read every response. Do not use this email to offer a discount — it signals that the product's value isn't worth the price, which increases future churn.

## Related Documents

- [Ops metrics](metrics.md)
- [Marketing launch strategy](../growth/marketing/launch-strategy.md)
- [Design ux-flows returning user](../experience/journeys/returning-user.md)
