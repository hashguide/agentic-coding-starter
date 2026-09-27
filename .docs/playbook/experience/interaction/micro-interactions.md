---
kind: playbook
domain: experience
topics: [experience, interaction, micro, interactions]
authority: guidance
---

# Micro-Interactions

Micro-interactions are the small, contained moments of motion and feedback that make UI feel alive and responsive. Each moment has a trigger, a defined duration, and an easing curve. They communicate state without requiring the user to read text.

## Catalog of Micro-Interaction Moments

### Button Press Feedback

**Trigger:** `mousedown` / `touchstart`

```css
.btn {
  transition: transform var(--duration-fast) var(--ease-in),
              background-color var(--duration-fast) var(--ease-default);
}

.btn:active {
  transform: scale(0.98);
  background-color: var(--color-primary-active);
}
```

- Duration: 100ms (fast)
- Easing: ease-in for press, ease-out for release
- Purpose: confirms the click was received

### Toggle Switch Animation

**Trigger:** User clicks to toggle

```css
.toggle-track {
  background-color: var(--color-neutral-300);
  transition: background-color var(--duration-base) var(--ease-default);
}

.toggle-track[data-checked='true'] {
  background-color: var(--color-primary);
}

.toggle-thumb {
  transform: translateX(0);
  transition: transform var(--duration-base) var(--ease-default);
}

.toggle-track[data-checked='true'] .toggle-thumb {
  transform: translateX(24px);
}
```

- Duration: 200ms (base)
- Easing: ease-default
- Purpose: thumb position communicates boolean state; color reinforces it

### Form Field Focus Transition

**Trigger:** Input receives focus

```css
.input {
  border-color: var(--color-border);
  box-shadow: none;
  transition: border-color var(--duration-fast) var(--ease-default),
              box-shadow var(--duration-fast) var(--ease-default);
}

.input:focus {
  border-color: var(--color-border-focus);
  box-shadow: 0 0 0 3px rgb(59 130 246 / 0.2);
  outline: none;
}
```

- Duration: 100ms (fast)
- Easing: ease-default
- Purpose: communicates which field is active; the focus ring is also an accessibility requirement

### Success Checkmark Animation

**Trigger:** Async operation completes successfully

```tsx
// Using Framer Motion for the SVG draw-on animation
import { motion } from 'framer-motion';

function SuccessCheckmark() {
  return (
    <motion.svg viewBox="0 0 24 24" fill="none" stroke="currentColor">
      <motion.path
        d="M5 13l4 4L19 7"
        strokeLinecap="round"
        strokeLinejoin="round"
        initial={{ pathLength: 0 }}
        animate={{ pathLength: 1 }}
        transition={{
          duration: 0.5,
          ease: [0.34, 1.56, 0.64, 1], // ease-spring
        }}
      />
    </motion.svg>
  );
}
```

- Duration: 500ms (deliberate)
- Easing: ease-spring (slight overshoot emphasizes the satisfaction of completion)
- Purpose: marks the transition from in-progress to done with a felt sense of completion

### Dropdown Open / Close

**Trigger:** Click on trigger element

```css
.dropdown-menu {
  opacity: 0;
  transform: translateY(-4px);
  transition: opacity var(--duration-base) var(--ease-out),
              transform var(--duration-base) var(--ease-out);
  pointer-events: none;
}

.dropdown-menu[data-open='true'] {
  opacity: 1;
  transform: translateY(0);
  pointer-events: auto;
}
```

- Open: 200ms ease-out (decelerates as it arrives)
- Close: 100ms ease-in (accelerates out of the way)
- Purpose: connects the trigger to the menu and provides spatial context

## Accessibility Note

All transitions must be disabled or shortened to near-zero for users with `prefers-reduced-motion: reduce`. See [motion.md](../visual/motion.md) for the global rule.

## Related Documents

- [Motion principles](../visual/motion.md)
- [Component states](component-states.md)
- [Optimistic UI](optimistic-ui.md)
