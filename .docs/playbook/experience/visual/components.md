---
kind: playbook
domain: experience
topics: [experience, visual, components]
authority: guidance
---

# Component States

Every interactive component must handle a defined set of states. An input that has no disabled state breaks when placed in a disabled form. A button with no loading state leaves the user uncertain whether their click registered. The state system is the minimum spec for every component.

## The State List

| State | Trigger | Visual treatment |
|---|---|---|
| **Default** | Initial render | Base styles — the component at rest |
| **Hover** | Cursor enters the component | Subtle background shift, shadow elevation, or color lightening |
| **Active / Pressed** | Mouse/touch down | Darker background, slight inward transform (scale-down or inset shadow) |
| **Focus** | Keyboard focus | High-contrast focus ring (2px, `--color-border-focus`) — never `outline: none` without replacement |
| **Disabled** | `disabled` attribute | Muted foreground and background, `cursor: not-allowed`, `aria-disabled="true"` |
| **Loading** | Async action in progress | Spinner inside or replacing content; `aria-busy="true"` |
| **Error** | Validation failure or operation failure | Error color border/background, error icon, error message below |
| **Success** | Operation completed | Success color treatment, checkmark icon, may auto-clear |
| **Empty** | No content to display | Empty state component (see [empty-states.md](../interaction/empty-states.md)) |

## Buttons

| State | Visual |
|---|---|
| Default | Primary fill, white text |
| Hover | Slightly darker fill (`--color-primary-hover`) |
| Active | Darker fill (`--color-primary-active`), `scale(0.98)` |
| Focus | Visible focus ring, same background as hover |
| Disabled | Muted neutral fill, muted text, `cursor: not-allowed` |
| Loading | Spinner replaces label (or appears alongside); width locked to prevent layout shift |

```tsx
function Button({ children, loading, disabled, onClick, ...props }: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled || loading}
      aria-busy={loading}
      className={cn('btn-primary', loading && 'btn-loading', disabled && 'btn-disabled')}
      {...props}
    >
      {loading ? <Spinner aria-hidden="true" /> : children}
    </button>
  );
}
```

## Inputs

| State | Visual |
|---|---|
| Default | Neutral border, placeholder text |
| Hover | Slightly stronger border |
| Focus | Primary-colored border (`--color-border-focus`), no outline suppression |
| Filled-valid | Default styling — valid is the quiet state |
| Filled-invalid | Red border, red text below describing the error |
| Disabled | Muted background, muted text, `cursor: not-allowed` |
| Submitting | Not individually disabled per field — the submit button enters its loading state |

Error messages below inputs are anchored to the specific field that caused the error. They appear on blur (when the user leaves the field), not on every keystroke.

## Cards

| State | Visual |
|---|---|
| Default | Surface background, subtle border or shadow |
| Hover (if interactive) | Elevated shadow, subtle background shift |
| Active (if selectable) | Primary border or filled left-border accent |
| Loading | Skeleton placeholder replacing content |
| Error | Error-colored border, error icon in content area |
| Empty | Empty state treatment if the card is a container |

## Modals

| State | Visual |
|---|---|
| Closed | Not rendered (or `display: none` + `aria-hidden`) |
| Opening | Entry transition (fade in + scale up, ~200ms) |
| Open — default | Content visible, backdrop overlay, focus trapped inside |
| Open — loading | Spinner inside, submit button in loading state, dismiss optional or locked |
| Open — error | Error message inside modal, form remains visible for correction |
| Open — destructive confirm | Red-accented confirm button, explicit warning text, cancel given equal prominence |
| Closing | Exit transition (fade out, ~150ms) |

```tsx
function Modal({ open, onClose, children }: ModalProps) {
  // Focus trap: keep keyboard focus inside modal while open
  const ref = useFocusTrap(open);

  return open ? (
    <div role="dialog" aria-modal="true" ref={ref}>
      <div className="modal-backdrop" onClick={onClose} />
      <div className="modal-panel">
        {children}
        <button onClick={onClose} aria-label="Close">×</button>
      </div>
    </div>
  ) : null;
}
```

## Related Documents

- [Design tokens](../brand/tokens.md)
- [Component states — forms, tables, modals](../interaction/component-states.md)
- [Empty states](../interaction/empty-states.md)
- [Frontend accessibility](../../frontend/accessibility.md)
