---
kind: playbook
domain: experience
topics: [experience, brand, identity]
authority: guidance
---

# Brand Identity

A brand's visual identity is its recognizable mark — the graphical element users associate with the product. Before designing screens, define what this mark is and establish rules for how it may be used.

## Logo Types

| Type | Definition | When to use |
|---|---|---|
| **Wordmark** | The product name rendered in a distinctive typeface | When the name itself is short, distinctive, and memorable (e.g., Google, Figma) |
| **Lettermark** | Initials of the product name as a typographic logo | When the full name is long but the initials are recognizable (e.g., IBM, HBO) |
| **Brandmark / Icon** | A standalone symbol or icon with no text | Used alongside the wordmark initially; stands alone once brand recognition is established |
| **Combination mark** | Wordmark + brandmark together | The safest starting point — works at all sizes, establishes the pairing |

Most early-stage products should use a combination mark. The brandmark alone is not recognizable until you have established it through repeated exposure.

## Clear Space

Clear space is the minimum empty area that must surround the logo on all sides. It prevents the logo from being crowded by surrounding elements.

**Rule:** Set the clear space minimum to the height of the capital letter in the wordmark (or the smallest unit in the brandmark grid). For example, if the "P" in the wordmark is 24px tall, the minimum clear space on each side is 24px.

Express this in your design tokens as a multiplier of the logo's base unit:

```css
--logo-clear-space: 1x; /* x = cap height or brandmark grid unit */
```

## Minimum Size

Define the minimum size at which the logo remains legible:
- **Digital:** 24px height for the icon-only mark; 120px width for the combination mark
- **Print:** 10mm height minimum

Below these sizes, switch to a simplified or icon-only version.

## Usage Rules

**Do:**
- Use the logo on approved backgrounds (light, dark, and any defined brand colors)
- Maintain proportions — resize uniformly
- Use the minimum clear space consistently

**Do not:**
- Stretch or distort the logo
- Recolor the logo outside the approved palette (primary on white, white on primary, and any other explicitly approved combinations)
- Apply drop shadows, outlines, or other effects not part of the defined variants
- Place the logo on backgrounds that reduce contrast below readable levels
- Use the logo smaller than the defined minimum sizes
- Rotate the logo

## Logo Files to Maintain

| File | Format | Use |
|---|---|---|
| `logo-combination-light.svg` | SVG | Light backgrounds, screens |
| `logo-combination-dark.svg` | SVG | Dark backgrounds |
| `logo-icon-only.svg` | SVG | Small UI contexts, favicons at large sizes |
| `favicon.ico` | ICO | Browser tab |
| `favicon.png` | PNG (32×32, 192×192, 512×512) | PWA, social sharing |

## Related Documents

- [Brand voice and tone](voice-and-tone.md)
- [Design tokens](tokens.md)
- [Branding overview](overview.md)
