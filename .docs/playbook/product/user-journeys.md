---
kind: playbook
domain: product
topics: [product, user, journeys]
authority: guidance
---

# User Journeys

A user journey maps what a specific persona does, thinks, and feels as they move through your product — from first awareness through the core workflow and beyond. Journeys reveal friction before you build it in, and they surface the emotional beats that distinguish a product people love from one they merely tolerate.

## Journey Mapping Template

Create one journey per primary persona. A journey is not a sitemap — it's a story with stages.

```markdown
## Journey: [Persona Name] — [Journey Name]

**Goal:** [What this persona is trying to accomplish by the end of this journey]
**Starting point:** [Where the persona is before the journey begins — what they know, what they're using now]

| Stage | Actions | Touchpoints | Emotions / Friction | Success state |
|---|---|---|---|---|
| [Stage name] | [What they do] | [What they interact with] | [How they feel; where they get stuck] | [What "done" looks like for this stage] |

**Journey success:** [Observable behavior that confirms the persona reached the journey's goal]
**Exit / failure state:** [What happens if the journey fails — where do they go?]
```

## Example Journey

```markdown
## Journey: Maya — First Invoice Sent

**Goal:** Send a professional invoice to a client within 10 minutes of opening the app for the first time.
**Starting point:** Maya clicked a signup link from a Twitter post. She has a client who needs an invoice today.

| Stage | Actions | Touchpoints | Emotions / Friction | Success state |
|---|---|---|---|---|
| Signup | Enters email + password | Signup form | Relieved it's short; anxious if it requires too much upfront | Account created, redirected to dashboard |
| Onboarding | Skips or completes name + business details | Onboarding wizard | Impatient if too many steps; uncertain what's required vs. optional | At least reaches the create-invoice page |
| Invoice creation | Adds client name, email, line items, amount | Invoice editor | Confident if the form is familiar; frustrated if fields are unclear | Invoice preview renders correctly |
| Send | Clicks "Send invoice" | Send button + email preview | Slightly anxious — "Will this look professional?" | Confirmation that the email was sent |
| Waiting | Monitors payment status | Dashboard or mobile | Checking frequently; frustrated if no status update | Client payment received; notification triggered |

**Journey success:** Maya sends her first invoice within 10 minutes of signup without contacting support.
**Exit / failure state:** Maya abandons if onboarding exceeds 5 steps, if the invoice editor has too many required fields, or if sending requires connecting an external email account.
```

## Cross-References

Every journey stage that involves a screen or flow should reference the relevant design documents once those documents are built. Update these references as documents are completed:

- Screen layouts → [design/layouts/](../experience/layouts/dashboard.md)
- UX flow details → [design/ux-flows/](../experience/journeys/new-user.md)
- Empty states → [design/interaction/empty-states.md](../experience/interaction/empty-states.md)
- Error feedback → [design/interaction/error-feedback.md](../experience/interaction/error-feedback.md)
- Notification touchpoints → [design/interaction/notifications.md](../experience/interaction/notifications.md)

## Related Documents

- [User personas](user-personas.md)
- [UX flow goals](../experience/journeys/flow-goals.md)
- [New user flow](../experience/journeys/new-user.md)
- [Returning user flow](../experience/journeys/returning-user.md)
