---
kind: playbook
domain: experience
topics: [experience, interaction, micro, content]
authority: guidance
---

# Micro-Content

Micro-content is the small-scale copy woven into the interface: button labels, navigation links, tooltips, placeholder text, empty state messages, error copy. Done well, it is invisible — users just know what to do. Done poorly, it is generic, confusing, or forces the user to stop and think.

## The Core Principle

Every piece of micro-content should be intentional and benefit-oriented. Generic labels are placeholders that ship to production without revision.

**Example:**

| Generic | Intentional |
|---|---|
| "About Us" | "How this works" or "Our approach" |
| "Learn More" | "See how pricing works" |
| "Submit" | "Create account" or "Save changes" |
| "Click here" | The verb describing what happens: "Download invoice" |
| "Error occurred" | "Couldn't save. Try again." |

The generic version works as a placeholder — it communicates approximately what something does. The intentional version tells the user specifically what will happen, what they'll get, or what problem will be solved.

## Before/After Table

| Element | Generic (avoid) | Intentional (use) |
|---|---|---|
| Signup button | "Sign Up" | "Get started free" or "Create your account" |
| Save button | "Save" | "Save changes" (or just "Save" when context is clear — be specific only when needed) |
| Delete confirm | "Are you sure?" | "Delete this project? This cannot be undone." |
| Empty state heading | "No data yet" | "You don't have any invoices yet" |
| Empty state CTA | "Add" | "Create your first invoice" |
| Nav link | "Features" | "What you can do" or "Features" (clear > clever) |
| Form placeholder | "Enter your email" | "name@company.com" (example format beats instruction) |
| Tooltip | "Help" | "This controls who can see your project" |
| Loading state | "Loading..." | "Generating your report..." |
| Success message | "Success!" | "Invoice sent to client@example.com" |

## Guidance

**Keep it short.** One sentence max for tooltips. Two to five words for button labels. Three words max for navigation items.

**Use active verbs.** "Download report" beats "Report download." Verbs create momentum; nouns create work.

**State the outcome, not the action.** "Save and publish" beats "Submit." "Start free trial" beats "Sign up."

**Specific beats clever.** A label that is clear and slightly boring is better than a label that is clever and ambiguous. Users do not stop to appreciate wordplay — they stop when they're confused.

**Do not address the user in third person.** "Users can access billing here" is documentation, not UI copy. "Access billing" or "View billing" is UI copy.

## Related Documents

- [Brand voice and tone](../brand/voice-and-tone.md)
- [Writing for UI](../content/writing-for-ui.md)
- [Onboarding copy](../content/onboarding-copy.md)
- [Error feedback](error-feedback.md)
- [Empty states](empty-states.md)
