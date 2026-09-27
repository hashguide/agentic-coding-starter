---
kind: playbook
domain: product
topics: [product, feature, prioritization]
authority: guidance
---

# Feature Prioritization

Every feature competes for the same finite resource: time. Prioritization frameworks exist to make that competition explicit and consistent, so "what gets built next" is a decision rather than a reflex. Use these frameworks at project start to scope the MVP and after launch to sequence improvements.

## RICE Scoring

RICE scores each feature on four dimensions, producing a single comparable number.

**Formula:** `Score = (Reach × Impact × Confidence) / Effort`

| Dimension | Definition | Example scale |
|---|---|---|
| **Reach** | How many users are affected in a given period | Number of users per month |
| **Impact** | How much does it move the key metric when a user encounters it | 3 = massive, 2 = high, 1 = medium, 0.5 = low, 0.25 = minimal |
| **Confidence** | How certain are you about the Reach and Impact estimates | 100% = high, 80% = medium, 50% = low |
| **Effort** | Person-months of work | Estimated total work |

**Worked example:**

| Feature | Reach | Impact | Confidence | Effort | Score |
|---|---|---|---|---|---|
| Email notifications on payment | 200 | 2 | 80% | 0.5 | 640 |
| CSV data export | 50 | 1 | 50% | 1 | 25 |
| Team member invites | 80 | 3 | 60% | 3 | 48 |

Email notifications score highest because it touches many users, has meaningful impact, and is low effort.

**When to use RICE:** When you have enough usage data to estimate Reach. Early pre-launch, the Reach estimate is speculative — use MoSCoW instead.

## MoSCoW

Categorize each feature into one of four buckets:

- **Must:** Without this, the product does not function or the user has no reason to pay.
- **Should:** High value, clearly needed soon, but the product can launch without it.
- **Could:** Nice to have; included if time allows; won't delay launch.
- **Won't:** Explicitly not in this version. (Saying "won't" is as important as saying "must.")

**Worked example — invoicing product:**

- **Must:** Create invoice, add line items, send by email, mark as paid
- **Should:** Invoice PDF download, payment reminders
- **Could:** Custom invoice templates, client portal
- **Won't (v1):** Recurring invoices, tax calculation, multi-currency

**When to use MoSCoW:** Pre-launch, when Reach data doesn't exist. Also useful for aligning stakeholders quickly — the "Won't" column prevents scope creep better than silence.

## Value vs. Effort Quadrant

Plot features on a 2×2 grid:

```
High value │ Quick wins ★   │  Major projects
           │ (do first)     │  (plan carefully)
───────────┼────────────────┼──────────────────
Low value  │ Fill-ins       │  Avoid
           │ (if time)      │  (don't build)
           └────────────────┴──────────────────
                Low effort       High effort
```

**When to use the quadrant:** For visual alignment in planning sessions or when explaining prioritization to a non-technical stakeholder. Not precise enough for solo ranking — combine with RICE or MoSCoW.

## When to Use Which Framework

| Situation | Recommended framework |
|---|---|
| Pre-launch MVP scope definition | MoSCoW |
| Post-launch backlog ranking with usage data | RICE |
| Visual team alignment meeting | Value vs. Effort quadrant |
| Rapid "should we build this?" decision | MoSCoW (one-minute gut-check version) |

## Related Documents

- [Project brief](project-brief.md)
- [Requirements](requirements.md)
- [Non-functional requirements](non-functional-requirements.md)
