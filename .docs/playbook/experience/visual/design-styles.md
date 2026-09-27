---
kind: playbook
domain: experience
topics: [experience, visual, design, styles]
authority: guidance
---

# Design Styles

A design style is the visual language of the interface — the set of aesthetic decisions that make it recognizable. Choosing a style should follow from the brand voice established in [voice-and-tone.md](../brand/voice-and-tone.md). A serious, expert-positioning brand rarely benefits from glassmorphism; a playful consumer brand rarely benefits from brutalism.

## Flat Design

**Characteristics:** No gradients, no drop shadows, no skeuomorphic depth. Clean geometric shapes, solid fills, strong color. Iconography is simple and outlined.

**Visual signal:** Modern, efficient, professional.

**Known for:** Early Google Material Design, Stripe dashboard, Figma interface.

**Use when:** Building a productivity or B2B tool where information density matters and decoration would distract.

## Minimal Design

**Characteristics:** Maximum white space, reduced color palette (often neutral with one accent), large typography, very few decorative elements.

**Visual signal:** Calm, focused, premium.

**Known for:** Linear, Vercel, Notion.

**Use when:** The product competes on clarity and focus. The content is the product.

## Glassmorphism

**Characteristics:** Frosted-glass translucency on panels (backdrop-filter: blur), layered depth, light borders, blurred backgrounds behind content layers.

**Visual signal:** Futuristic, sleek, visually rich.

**Known for:** macOS Big Sur system UI, some fintech dashboards.

**Use when:** The product has a strong visual identity and dark-mode-first design. Use sparingly — full-screen glassmorphism is visually heavy and can obscure readability.

**Caution:** `backdrop-filter: blur()` is GPU-intensive and affects performance on lower-end devices. Test on non-pro hardware before committing.

## Brutalist Design

**Characteristics:** Raw, intentional rule-breaking — visible structure, monospaced or heavy typefaces, high contrast, unconventional layouts, exposed grid lines or borders.

**Visual signal:** Opinionated, irreverent, distinctive.

**Known for:** Gumroad (early), some indie maker tools.

**Use when:** The brand is explicitly counter-establishment and differentiation from polished SaaS aesthetics is the point. Rarely appropriate for enterprise or compliance-adjacent products.

## Neumorphic Design

**Characteristics:** Soft shadows creating raised/inset tactile effects from the background, monochromatic palette, subtle depth.

**Visual signal:** Soft, satisfying, futuristic-tactile.

**Known for:** Niche UI design showcases, some music/audio app interfaces.

**Use when:** The product is primarily visual/entertainment and the tactile feel is product-relevant (e.g., audio equipment controls). Rarely appropriate for information-dense dashboards — the soft shadows reduce contrast and fail WCAG AA.

**Caution:** Neumorphic designs frequently fail accessibility contrast requirements. Verify against WCAG AA before using.

## Skeuomorphic Design

**Characteristics:** UI elements mimic their physical counterparts (leather textures, wooden shelves for bookshelves, realistic toggle switches).

**Visual signal:** Familiar, approachable, analog-feeling.

**Known for:** Early iOS (pre-iOS 7), early macOS apps.

**Use when:** The user's mental model is strongly tied to the physical analog being replaced (e.g., a music instrument app, an analog meter visualization). Largely deprecated in general SaaS interfaces.

## Choosing a Style

| Brand voice characteristic | Likely style fit |
|---|---|
| Expert, efficient, professional | Flat or Minimal |
| Premium, calm, focused | Minimal |
| Futuristic, tech-forward | Glassmorphism or Flat |
| Playful, irreverent, indie | Brutalist |
| Soft, consumer, personal | Neumorphic (with care) or Flat |

The style should reinforce, not contradict, the brand voice. If the voice is direct and no-nonsense but the interface is decorated with soft shadows and animated blobs, the aesthetic fights the writing.

## Related Documents

- [Brand voice and tone](../brand/voice-and-tone.md)
- [Design tokens](../brand/tokens.md)
- [Design system color](color.md)
