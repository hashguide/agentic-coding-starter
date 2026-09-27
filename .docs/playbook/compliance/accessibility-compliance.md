---
kind: playbook
domain: compliance
topics: [compliance, accessibility]
authority: guidance
---

# Accessibility Compliance

Web accessibility is both a legal requirement and a product quality factor. Products that are inaccessible exclude users with disabilities — and in some jurisdictions, face legal liability for doing so.

## ADA (United States)

The Americans with Disabilities Act (ADA) applies to places of public accommodation. US courts have increasingly found that websites and web applications operate as places of public accommodation, making ADA Title III applicable. While there is no single technical standard cited in the ADA, courts and the Department of Justice have consistently referenced WCAG 2.1 AA as the applicable standard.

ADA litigation risk is real: thousands of website accessibility lawsuits are filed annually in the US. Any publicly accessible SaaS product faces this risk.

## WCAG 2.1 AA — The Practical Target

WCAG 2.1 Level AA is the standard that satisfies both ADA and EAA requirements. It organizes requirements under four principles:

**Perceivable:** Information must be presentable in ways users can perceive.
- Text alternatives for all non-text content
- Captions for video; audio descriptions where needed
- Color contrast ratio ≥ 4.5:1 for body text, ≥ 3:1 for large text
- Content doesn't rely on color alone to convey meaning

**Operable:** Interface components must be operable.
- All functionality accessible via keyboard
- No keyboard traps
- Sufficient time to read and use content
- No content that flashes more than three times per second (seizure risk)
- Skip navigation links for keyboard users
- Descriptive page titles and focus indicators

**Understandable:** Information and UI operation must be understandable.
- Text is readable and understandable (language declared in HTML)
- Pages behave predictably
- Input assistance: labels, instructions, error identification and suggestions

**Robust:** Content must be interpretable by assistive technologies.
- Valid HTML
- Name, role, value exposed for all UI components
- Status messages exposed to assistive technologies without focus

**Practical minimum:** Ensure keyboard navigation works for all interactive elements, all images have meaningful alt text, form inputs have labels, error messages are associated with the relevant field, and focus is managed correctly in modals and dialogs. See [architecture/frontend/accessibility.md](../frontend/accessibility.md) for implementation guidance.

## European Accessibility Act (EAA)

The EAA came into force in June 2025 (with enforcement for existing products from June 2025 onward). It applies to products and services offered in the EU, including web applications and mobile apps.

**Who it applies to:** Any business offering digital products or services to EU consumers. Small enterprises (fewer than 10 employees AND annual turnover ≤ €2 million) are exempt.

**Technical standard:** EN 301 549, which incorporates WCAG 2.1 AA. Products compliant with WCAG 2.1 AA are compliant with the EAA's technical requirements.

**Conformance statement:** The EAA requires an accessibility statement on the product or website declaring the conformance level, known exceptions, and a contact mechanism for reporting accessibility issues.

## Testing for Compliance

1. **Automated testing:** Axe-core, Lighthouse, or Pa11y catch ~30–40% of WCAG issues automatically. Add to CI.
2. **Keyboard testing:** Tab through every screen. Verify focus is visible, no traps exist, all actions are reachable.
3. **Screen reader testing:** Test with VoiceOver (macOS/iOS) and NVDA or JAWS (Windows) for critical flows.
4. **Manual audit:** Go through the WCAG 2.1 AA checklist for each major screen.

## Related Documents

- [Frontend accessibility](../frontend/accessibility.md)
- [Design color system](../experience/visual/color.md)
- [GDPR compliance](gdpr.md)
- [Security checklist](../security/checklist.md)
