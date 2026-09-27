---
kind: playbook
domain: product
topics: [product, project, brief]
authority: guidance
---

# Project Brief

Before writing a line of code, lock down what you're actually building. The project brief is the output of three exercises: a brain dump that captures every idea, an AI-assisted discovery interview that distills those ideas into a defensible scope, and an MVP lock step that produces a single, concrete document others can build against. Skipping this step is how projects become shapeless and never ship.

Before selecting architecture or monetization, classify how the product delivers value using [product type selection](product-type-selection.md). Do not assume the result needs accounts, subscriptions, multi-tenancy, or a continuously hosted web application.

## The Brain Dump Exercise

Set a 20-minute timer. Write down every feature, use case, user type, integration, and capability you imagine for the product. Do not filter. The goal is to exhaust the idea space so you can choose deliberately rather than remember things mid-build.

After the timer: group items by theme. Circle the three to five that are load-bearing — without them, the product has no reason to exist.

## The AI Discovery Interview

Paste your brain dump into a conversation with an AI assistant. Use this five-question format:

1. **What problem does this solve, and for whom?** Force a single sentence. "It helps [specific person] do [specific thing] without [current pain]."
2. **What does success look like for the user in the first 5 minutes?** Not features — the actual moment of value delivered.
3. **What is the one action the product must enable for a user to be willing to pay?** This is the Payment Gate.
4. **What would make this different from the three nearest alternatives a user would find in a 10-minute search?** Must be a specific, defensible difference — not "better UX."
5. **What can be cut from the brain dump without undermining the core value?** The answer to this defines your first post-MVP roadmap.

Save the AI's responses. They become the first section of the project brief.

## MVP Scope Lock

Three gates. If a feature doesn't pass all three, it's not in the MVP.

**Single Core Workflow:** Is this feature part of the one workflow that delivers the core value? If the user can accomplish the primary task without it, cut it.

**Payment Gate:** Does this feature sit between the user and their first willingness to pay? If it doesn't, it's a post-launch feature.

**Launch Success Criteria:** State three measurable conditions that, if true after 30 days, mean the launch succeeded. Examples: "10 paying customers," "less than 20% churn in week one," "at least 3 users complete the core workflow without support." Features that don't affect any success criterion get cut.

## Monetization Model Comparison

| Model | Best fit | Revenue predictability | Upfront complexity |
|---|---|---|---|
| Flat subscription | Fixed feature set, recurring use | High | Low |
| Usage-based | Variable consumption, API-style products | Medium | Medium |
| Freemium-to-paid | Network-effect products, viral growth loops | Low early | Medium |
| One-time purchase | Tools, assets, templates with no ongoing service | Low | Low |
| Seat-based | Team products, per-user value scaling | High | Medium |

For most solo-built SaaS products launching to individuals: start with a flat subscription (one or two tiers). Usage-based billing requires accurate metering infrastructure that takes time to build correctly. See [monetization overview](../business/monetization/overview.md) for the full model analysis.

## PROJECT_BRIEF.md Template

Produce this file and commit it to the repository root before writing any code.

```markdown
# PROJECT_BRIEF.md

## Problem Statement
[One sentence: who, what pain, what outcome]

## Target User
[Specific persona description — see core/user-personas.md for the full persona template]

## Core Value (Payment Gate)
[The one action the product enables that makes a user willing to pay]

## Differentiation
[Specific, defensible difference from the three nearest alternatives]

## MVP Feature List
[Bulleted list — only features that passed all three scope lock gates]

## Out of Scope (v1)
[Explicit list of cut features, with a one-line rationale for each cut]

## Launch Success Criteria
- [ ] [Measurable condition 1 — 30-day target]
- [ ] [Measurable condition 2 — 30-day target]
- [ ] [Measurable condition 3 — 30-day target]

## Monetization Model
[Selected model and tier structure]

## Open Questions
[Unanswered assumptions that require user research or early feedback to resolve]
```

## Related Documents

- [User personas](user-personas.md)
- [Feature prioritization](feature-prioritization.md)
- [Monetization overview](../business/monetization/overview.md)
- [Non-functional requirements](non-functional-requirements.md)
- [Product type selection](product-type-selection.md)
