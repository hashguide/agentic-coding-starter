---
kind: playbook
domain: experience
topics: [experience, content, onboarding, copy]
authority: guidance
---

# Onboarding Copy

Onboarding copy guides a new user from account creation to their first meaningful action. It is the most consequential UI copy in the product — users who successfully onboard activate; users who don't, churn. The tone is encouraging, the sentences are short, and every piece of copy is action-oriented.

## Tone Guide for Onboarding

Onboarding copy is the warmest version of the brand voice. The user is new, potentially unsure, and looking for direction. The copy should feel like a confident, friendly guide — not a tutorial manual.

**Characteristics:**
- **Encouraging:** Acknowledge the user is in the right place and has made a good start
- **Concise:** No paragraphs. Short sentences or fragments. The user is reading to move forward, not to learn.
- **Action-oriented:** Every prompt ends with a clear next step or a button
- **Specific:** Name the specific thing to do, not a general category

**What to avoid:**
- Hollow affirmations: "Amazing! Great job! You're crushing it!" — patronizing on second use
- Jargon before the user knows the product: "Create a workspace environment" when "Set up your first project" is clearer
- Long explanations: Explain in the flow, not in a paragraph before the flow

## Tooltip Copy

Tooltips in an interactive product tour should be one sentence maximum.

| Too long | Right length |
|---|---|
| "This button will open the project creation dialog where you can specify a name, description, visibility settings, and initial team members." | "Create your first project here." |
| "The notifications panel shows all recent activity across your account and your team's work." | "See recent activity for you and your team." |

One sentence. Noun + verb + outcome. Done.

## Checklist Item Phrasing

Checklist items should state the action and the benefit together:

| Just the action (weak) | Action + benefit (better) |
|---|---|
| "Create a project" | "Create a project — organize your work" |
| "Invite a team member" | "Invite a team member — collaborate together" |
| "Connect Stripe" | "Connect Stripe — start accepting payments" |
| "Set up notifications" | "Set up notifications — never miss an update" |

The benefit doesn't need to be long. A short phrase after an em dash is enough. The user should understand why they're doing this step, not just that it's on the list.

## Welcome Messages

The first message a user sees after signup sets the tone. Make it specific and human:

```
Welcome to [Product], [first name].

You're all set. Here's what most people do first:
→ [Primary onboarding action]

It takes about [N] minutes.
```

Not: "Welcome! We're so excited to have you here! There are lots of great features to explore! Let's get started on your journey!"

## Progress Messages

As the user completes onboarding steps, confirm each completion briefly:

- "Profile created." (not "Great job creating your profile!")
- "Stripe connected. You can now accept payments."
- "Team member invited. They'll get an email shortly."

The second sentence explains what the completed step enables — not just that it was completed.

## Related Documents

- [Brand voice and tone](../brand/voice-and-tone.md)
- [Onboarding layout](../layouts/onboarding.md)
- [New user flow](../journeys/new-user.md)
- [Micro-content](../interaction/micro-content.md)
