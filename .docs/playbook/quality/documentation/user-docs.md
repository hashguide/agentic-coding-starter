---
kind: playbook
domain: quality
topics: [quality, documentation, user, docs]
authority: guidance
---

# User Documentation

User documentation helps users succeed with the product without contacting support. Well-written user docs reduce support volume, improve retention, and help users discover features they didn't know existed.

## The Three-Audience Model

Different user types need different documentation:

| Audience | What they need | Where to find them |
|---|---|---|
| **New users** | How to get started quickly, reach their first success | Getting started guide, onboarding checklist |
| **Intermediate users** | How to use specific features in depth | Feature reference, tutorials |
| **Power users** | Advanced configuration, edge cases, integrations | Advanced guides, API reference |

Write with the new user in mind first. Advanced users can read ahead; new users get lost if you skip foundations.

## Tutorial Format Template

A tutorial guides a user through completing a specific task and reaching a concrete outcome.

```markdown
# [Tutorial Title]

**What you'll accomplish:** One sentence describing the outcome.

**Prerequisites:** What the user needs before starting.

**Time estimate:** 10–15 minutes

---

## Step 1 — [Step title]

[One or two sentences describing the action and why.]

[Screenshot or illustration if the UI is not obvious]

[Specific action: "Click the **Save** button in the top right corner."]

## Step 2 — [Step title]

...

---

## Troubleshooting

**Problem: [Specific thing that might go wrong]**
Solution: [Exact steps to resolve it]

---

## Next Steps

- Related tutorial
- Feature reference for this area
```

**Tutorial rules:**
- The outcome must be testable — the user must be able to verify they succeeded
- Every step uses the exact UI label (bold): "Click **Save**," not "save it"
- Screenshots show the exact state of the UI before the action, not after
- Troubleshooting section covers the top 2–3 failure modes

## Changelog Entry Format

Changelog entries are written from the user's perspective — what changed for them, not what changed in the code.

**Template:**

```markdown
## [Month Day, Year]

### New
- **Feature Name**: Brief description of what it does and why it's useful.
  [Optional: link to tutorial or documentation]

### Improved
- **Feature Name**: What got better. Example: "Export now completes in seconds instead of minutes."

### Fixed
- **Issue description from user's perspective.** Example: "Fixed a bug where notifications 
  were not marked as read after clicking them."
```

**What not to write in a changelog:**
- Technical implementation details ("Refactored the export worker to use streaming")
- Internal ticket numbers ("Fixes #1234")
- Vague entries ("Performance improvements", "Bug fixes")

Write as if explaining to a user what changed in their experience, not what changed in the code.

## In-App Help

Three types of in-app help, each with a specific job:

| Type | Format | Length | When to show |
|---|---|---|---|
| **Tooltip** | Single sentence | Max 80 characters | On hover over a label or icon |
| **Helper text** | 1–2 sentences below a form field | Max 120 characters | Always visible in complex forms |
| **Empty state** | Explanation + CTA | 2–3 sentences | When a list/section has no content |

In-app help must be actionable. "Add your first project to get started" is better than "No projects found."

## Support-to-Docs Pipeline

Every recurring support question is a documentation gap. Track support themes:
- Week 1: "How do I export my data?" → Write the export tutorial
- Week 2: "Why is my invoice wrong?" → Add billing FAQ section
- Week 4: "Can I change my email?" → Add account settings guide

Support volume is the most reliable signal for what documentation to write next. Don't write documentation you predict users will want — write documentation users demonstrably need.

## Related Documents

- [Docs developer documentation](dev-docs.md)
- [Design content writing for UI](../../experience/content/writing-for-ui.md)
- [Design content onboarding copy](../../experience/content/onboarding-copy.md)
