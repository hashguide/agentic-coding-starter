---
kind: playbook
domain: frontend
topics: [frontend, accessibility]
authority: guidance
---

# Frontend Accessibility

Accessibility is not a polish item — it's a correctness requirement. A product that keyboard users, screen reader users, or users with visual impairments cannot use has a defect. WCAG 2.1 AA is the practical target that satisfies both legal compliance baselines (ADA, European Accessibility Act) and good engineering practice.

## WCAG AA Baseline Checklist

### Keyboard Navigation

- [ ] Every interactive element (links, buttons, form fields, modals, menus) is reachable via Tab
- [ ] Tab order follows logical reading order — not determined by DOM position alone when CSS reorders visually
- [ ] Keyboard shortcuts (if any) do not conflict with browser or screen reader shortcuts
- [ ] Focus is visible at all times — never suppressed with `outline: none` without a custom visible alternative

### Visible Focus Indicators

- [ ] Every focusable element has a visible focus indicator (the browser default is acceptable; removing it without replacement is not)
- [ ] Focus indicator has at least a 3:1 contrast ratio against adjacent colors

### Color Contrast

- [ ] Normal text (< 18pt / < 14pt bold): 4.5:1 contrast ratio against background
- [ ] Large text (≥ 18pt / ≥ 14pt bold): 3:1 contrast ratio against background
- [ ] UI components and graphical objects: 3:1 against adjacent colors
- [ ] Verify with: [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) or browser DevTools

### ARIA Labels

- [ ] Icon-only buttons have `aria-label` describing the action ("Close", "Delete notification", "Mark all as read")
- [ ] Images that convey information have `alt` text; decorative images have `alt=""`
- [ ] Form fields have associated `<label>` elements (not just placeholder text)
- [ ] Landmark regions (`<header>`, `<main>`, `<nav>`, `<footer>`) are used correctly

### Color as the Only Signal

- [ ] Error states are not communicated by color alone — pair red color with an error icon or text
- [ ] Status indicators include text or icons, not just color coding

## Focus Trapping in Modals

Modals must trap focus — keyboard navigation must not escape to the page behind the modal while it is open.

```typescript
// React — use a focus trap library or implement manually
import { useEffect, useRef } from 'react';

function Modal({ isOpen, onClose, children }: ModalProps) {
  const modalRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!isOpen) return;
    const modal = modalRef.current;
    if (!modal) return;

    // Move focus into the modal
    const firstFocusable = modal.querySelector<HTMLElement>(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    firstFocusable?.focus();

    // Trap focus within the modal
    function handleKeyDown(e: KeyboardEvent) {
      if (e.key === 'Escape') { onClose(); return; }
      if (e.key !== 'Tab') return;

      const focusable = Array.from(
        modal!.querySelectorAll<HTMLElement>(
          'button:not([disabled]), [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
        )
      );
      const first = focusable[0];
      const last = focusable[focusable.length - 1];

      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault();
        last.focus();
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault();
        first.focus();
      }
    }

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div role="dialog" aria-modal="true" ref={modalRef}>
      {children}
    </div>
  );
}
```

## Security

**Focus trapping correctness:** An incorrectly implemented focus trap can leak focus to content behind the modal, allowing keyboard users to interact with elements that should be inactive. Use a tested focus trap library (e.g., `focus-trap-react`) for correctness.

**tabindex misuse:** `tabindex` values greater than 0 create an explicit tab order that overrides the natural DOM order. This often creates keyboard traps or unintuitive navigation paths. Use `tabindex="0"` to make an element focusable, or `tabindex="-1"` to make it programmatically focusable but excluded from tab order. Never use positive `tabindex` values.

## Related Documents

- [Compliance accessibility](../compliance/accessibility-compliance.md)
- [Design system components](../experience/visual/components.md)
- [Frontend rendering](rendering.md)
