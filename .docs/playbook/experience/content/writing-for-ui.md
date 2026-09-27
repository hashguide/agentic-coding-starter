---
kind: playbook
domain: experience
topics: [experience, content, writing, for, ui]
authority: guidance
---

# Writing for UI

UI copy is the copy users interact with while using the product — button labels, error messages, empty states, tooltips, confirmation dialogs. Good UI copy is invisible: it answers the user's implicit question ("what does this do?", "what happened?", "what do I do next?") without requiring them to stop and think.

## Voice-Consistent Writing Checklist

Before shipping any UI copy, check it against the brand voice established in [voice-and-tone.md](../brand/voice-and-tone.md):

- [ ] **Short:** Can this be said in fewer words without losing meaning? Cut the extra words.
- [ ] **Active voice:** "Delete project" beats "Project will be deleted." Subject acts; don't use passive.
- [ ] **Specific over generic:** "Create your first invoice" beats "Get started." Name the specific thing.
- [ ] **Verb-first for actions:** Buttons and CTAs start with a verb: "Save", "Delete", "Send invoice", "Start trial".
- [ ] **Present tense:** "Saving..." not "Changes will be saved." "Sent" not "Your email was sent."
- [ ] **Consistent with voice:** Does this sound like the rest of the product?

## Common UI Copy Mistakes

| Mistake | Why it fails | Fix |
|---|---|---|
| "Submit" button | Generic — doesn't say what happens | "Save changes", "Create account", "Send invoice" |
| "Click here" link | Doesn't describe the destination | The linked text IS the label: "View your invoices" |
| "Something went wrong" | Provides no actionable information | "Couldn't save. Check your connection and try again." |
| "Are you sure?" confirm | Doesn't state what the action is | "Delete 'Project Alpha'? This cannot be undone." |
| "Error occurred" | Useless — obviously something failed | Specify what failed and what to do |
| "Please try again later" | Vague time + no reason | "Couldn't reach the server. Try again in a few seconds." |
| "Invalid input" | Doesn't say which field or why | "Must be a valid email address" |
| "Your data has been successfully saved" | Wordy and bureaucratic | "Saved" or "Changes saved" |

## Error Messages

Error messages have a formula: what went wrong + how to fix it.

```
Bad:  "Error: 422"
Bad:  "Something went wrong"
Bad:  "Invalid email or password"  ← too vague (also doesn't follow the auth rule)
Good: "Incorrect email or password."  ← specific (and follows the auth enumeration rule)
Good: "Couldn't save your changes. Check your connection and try again."
Good: "Name must be between 1 and 100 characters."
```

## Empty State Copy

Three-part template: context + explanation + action.

```
[Icon]
"You don't have any invoices yet."
[Create invoice →]
```

Not: "No invoices found." — passive, cold, doesn't explain or help.

## Confirmation Dialogs

State the specific action and consequence:

```
Delete "Invoice #2045"?

This will permanently remove the invoice and all associated line items.
This cannot be undone.

[Cancel]  [Delete invoice]
```

Not: "Are you sure?" — doesn't tell the user what they're confirming.

## Related Documents

- [Brand voice and tone](../brand/voice-and-tone.md)
- [Micro-content](../interaction/micro-content.md)
- [Error feedback](../interaction/error-feedback.md)
- [Empty states](../interaction/empty-states.md)
