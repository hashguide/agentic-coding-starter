---
kind: playbook
domain: experience
topics: [experience, brand]
authority: guidance
---

# Branding Overview

Branding and visual design are often treated as the same thing. They are not, and conflating them causes the most common early-stage design mistake: building screens before decisions about identity and voice are made, then discovering three months in that the visual language is inconsistent with the product's positioning.

## Branding vs. Visual Design System

**Branding** is the set of decisions that define how the product presents itself to the world:
- **Identity:** Logo, wordmark, color palette, visual personality
- **Voice:** How the product sounds in written copy, error messages, onboarding prompts, marketing
- **Positioning:** What the product is, who it's for, and why it's different

**Visual design system** is the implementation of branding decisions in interface components:
- Design tokens: specific color values, type sizes, spacing units, motion durations
- Components: buttons, inputs, cards, modals — each with defined states
- Patterns: forms, navigation, data display conventions

The relationship is directional: branding decisions determine what the design system looks like. If you build a design system before making branding decisions, you will rebuild it when those decisions are made. The tokens are just numbers until branding gives them meaning.

## Why Order Matters

A product that builds UI before settling on voice ends up with copy that ranges from formal to casual depending on who wrote each screen. A product that starts coding screens before the logo is decided may commit to an accent color that clashes with the eventual brand palette.

Getting branding right before UI work begins costs 1–2 days. Retrofitting brand decisions onto finished screens costs weeks.

## The Documents in This Directory

- [identity.md](identity.md) — Logo types, usage rules, clear space
- [voice-and-tone.md](voice-and-tone.md) — Voice spectrum, tone shifts by context
- [tokens.md](tokens.md) — Complete design token system (color, spacing, type, motion)

## Related Documents

- [Design system components](../visual/components.md)
- [Voice-and-tone guidelines](voice-and-tone.md)
- [Design styles](../visual/design-styles.md)
