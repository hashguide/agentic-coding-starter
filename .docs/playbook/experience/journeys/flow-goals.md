---
kind: playbook
domain: experience
topics: [experience, journeys, flow, goals]
authority: guidance
---

# Flow Goals

A user flow without a defined goal is a sequence of screens, not a design. Before laying out any screens, state the flow's specific, measurable outcome. The goal determines which screens belong in the flow, in what order, and what the success state looks like.

## The Principle

Every flow has one specific, measurable goal. Not "users can sign up" — that describes a capability. A goal describes an outcome:

> "Goal: The visitor completes account creation and lands in the product ready to take the first meaningful action."

Or:

> "Goal: The returning user who received a 'payment failed' notification resolves the issue and resumes their subscription within 3 minutes."

A specific goal makes design decisions obvious. If a screen doesn't move the user toward the stated goal, it doesn't belong in this flow.

## Flow Goal Template

Before designing a new flow, complete this template:

```
Flow name: [descriptive name]
Trigger: [what causes the user to enter this flow]
Starting state: [where the user is and what they know/have when the flow begins]
Goal: [the specific, measurable outcome — what must be true for the flow to be "done"]
Success state: [where the user lands and what they see when the goal is achieved]
Failure states: [what happens if the user drops out or encounters an error]
```

**Example — Signup Flow:**
```
Flow name: New user signup
Trigger: Visitor clicks "Get started free" on the landing page
Starting state: Anonymous visitor with no account
Goal: Visitor creates an account and lands in the product with the first prompt to take action
Success state: Authenticated user on the empty dashboard, with onboarding checklist visible
Failure states: (1) Email already registered → same "check your email" response (no enumeration)
               (2) Network error → inline error, form remains filled
               (3) Abandoned → email capture for re-engagement if partial signup
```

## Why This Matters

Without a stated goal, flows accumulate screens that seem reasonable individually but collectively delay the user from reaching value. "Let's also ask for their company name" is only a reasonable addition if the company name is used in the success state. If it isn't, it's friction that serves internal data collection, not the user's goal.

The goal also defines the flow's scope boundary: if a screen falls outside the path from trigger to success state, it belongs in a different flow.

## Related Documents

- [New user flow](new-user.md)
- [Returning user flow](returning-user.md)
- [Onboarding layout](../layouts/onboarding.md)
