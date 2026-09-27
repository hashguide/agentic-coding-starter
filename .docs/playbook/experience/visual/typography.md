---
kind: playbook
domain: experience
topics: [experience, visual, typography]
authority: guidance
---

# Typography

Typography is the primary carrier of content in a product interface. A well-chosen type scale creates clear hierarchy; readable body text reduces cognitive load; consistent pairings reinforce the brand's visual identity.

## Type Scale

Use the scale defined in [design tokens](../brand/tokens.md) as the source of truth. The following is the reference for applying each size:

| Token | Size / Line-height | Role |
|---|---|---|
| `text-xs` | 12px / 16px | Captions, metadata, timestamps |
| `text-sm` | 14px / 20px | Labels, helper text, secondary information |
| `text-base` | 16px / 24px | Body text — default for prose and UI descriptions |
| `text-lg` | 18px / 28px | Lead paragraph, introductory text |
| `text-xl` | 20px / 28px | Section headings (H3/H4 in content) |
| `text-2xl` | 24px / 32px | Page headings (H2 in content) |
| `text-3xl` | 30px / 36px | Display headings (H1 in content) |
| `text-4xl` | 36px / 40px | Hero headlines |

## Font Pairing

Three roles to fill, ideally with two or three typefaces:

| Role | Purpose | Example fonts |
|---|---|---|
| **Display face** | Headlines, large marketing text | Cal Sans, Sora, Plus Jakarta Sans, Outfit |
| **Body face** | All UI text at `text-base` and below | Inter, DM Sans, Geist, Nunito Sans |
| **Mono face** | Code blocks, technical values, timestamps | JetBrains Mono, Fira Code, Geist Mono |

**Pairing rule:** The display face and body face should contrast in personality without clashing. A rounded, humanist display face pairs well with a geometric, clean body face. Avoid pairing two faces that are nearly identical — the difference is imperceptible and adds load time.

**Load only weights you use.** Loading six font weights for a variable font that uses three increases page weight unnecessarily. Common weights needed: 400 (regular), 500 (medium), 600 (semibold), 700 (bold).

## Readability Rules

**Maximum line length:** 60–75 characters for body text. Lines longer than 80 characters increase reading fatigue. Use `max-width: 65ch` on content columns.

**Minimum body text size:** 16px (`text-base`). Avoid 12px or 14px for anything that users are expected to read in quantity — use these sizes only for secondary metadata.

**Contrast requirements (WCAG AA):**
- Body text (< 18px regular or < 14px bold): contrast ratio ≥ 4.5:1 against background
- Large text (≥ 18px regular or ≥ 14px bold): contrast ratio ≥ 3:1 against background
- Interactive elements (buttons, links): ≥ 4.5:1

**Line height:** Use the token values. Body text at 16px should have a line-height of 24px (1.5×). Heading text at large sizes can use tighter line-heights (1.1–1.3×) because users scan rather than read.

**Letter spacing:** Use the font's default letter spacing for body text. Display sizes may benefit from slightly negative tracking (`letter-spacing: -0.02em`) for tighter, more polished headlines.

## Common Mistakes

- Using more than three distinct font weights in one design (creates visual noise)
- Setting body text below 16px for non-supplementary content
- Using a display face for long-form body text (display faces are optimized for large sizes, not reading)
- Skipping the mono face — rendered code in a sans-serif body font is harder to scan

## Related Documents

- [Design tokens](../brand/tokens.md)
- [Design system color](color.md)
- [Accessibility compliance](../../compliance/accessibility-compliance.md)
