---
kind: playbook
domain: experience
topics: [experience, interaction, component, states]
authority: guidance
---

# Component States

Forms, tables, and modals each have distinct states that must be handled explicitly. An unhandled state is a blank or broken-looking UI that users will encounter because the network is slow, data is absent, or an error occurred.

## Form Field States

| State | Trigger | Visual / behavior |
|---|---|---|
| **Empty** | Initial render, no value | Placeholder text visible; default border |
| **Filled-valid** | User has typed a valid value | No special treatment — valid is the quiet state |
| **Filled-invalid** | User has entered an invalid value and blurred | Red border; red error message anchored below the field; `aria-invalid="true"` |
| **Submitting** | Form submit in progress | Submit button enters loading state; fields are not individually disabled (keep text visible and selectable) |
| **Submitted-success** | Server confirms success | Success feedback — toast or inline success message; form may reset or redirect |
| **Submitted-error** | Server returns an error | Error message displayed; form remains visible for correction |

**Error timing:** Validate on blur (when user leaves the field), not on every keystroke. Triggering on every keystroke punishes the user for being mid-input.

**Submit button:** Transitions to loading state on submit. Does not re-enable until the request resolves. Prevents double-submit.

## Data Table / List States

| State | Trigger | Treatment |
|---|---|---|
| **Loading** | Initial fetch, page change | Skeleton rows at the expected column widths — not a spinner replacing the entire table |
| **Empty** | Query returns zero results | Proper empty state with explanation and CTA — see [empty-states.md](empty-states.md) |
| **Error** | Fetch fails | Error message with a retry action inline in the table area |
| **Partial / paginating** | More pages available | Pagination controls: page numbers, previous/next; or "Load more" button |
| **Filtered-no-results** | Active filter returns zero rows | Distinct from the "empty" state — message explains the filter returned nothing and offers to clear filters |

The filtered-no-results state must not look identical to the true empty state. "No projects match your search" + "Clear search" is different from "You don't have any projects yet" + "Create project."

## Modal States

| State | Trigger | Treatment |
|---|---|---|
| **Closed** | Default | Not rendered, or `display: none` + `aria-hidden="true"` |
| **Opening** | Trigger click | Entry transition: fade in backdrop + panel scale (0.95→1), ~300ms ease-out; focus moves into modal |
| **Open — default** | Transition complete | Content visible; backdrop overlay; focus trapped inside modal; Escape closes |
| **Open — loading (inside)** | User submits form within modal | Submit button in loading state; content dims or shows spinner; modal not dismissible during operation |
| **Open — error (inside)** | Operation fails | Error message inside modal below content; form remains visible for retry; modal remains open |
| **Open — destructive confirm** | User triggers a destructive action | Red-accented confirm button; explicit warning text stating what will be deleted and that it cannot be undone; cancel button given equal size/prominence |
| **Closing** | Escape, backdrop click, or programmatic close | Exit transition: fade out, ~200ms ease-in; focus returns to trigger element |

**Destructive confirm pattern:**

```tsx
function DeleteProjectModal({ project, open, onClose, onConfirm }: DeleteModalProps) {
  const [loading, setLoading] = React.useState(false);

  async function handleConfirm() {
    setLoading(true);
    try {
      await onConfirm();
      onClose();
    } catch {
      setLoading(false);
      // Error state handled by parent or inline
    }
  }

  return (
    <Modal open={open} onClose={loading ? undefined : onClose}>
      <h2>Delete "{project.name}"?</h2>
      <p>This will permanently delete the project and all its data. This cannot be undone.</p>
      <div className="modal-actions">
        <button onClick={onClose} disabled={loading}>Cancel</button>
        <button
          onClick={handleConfirm}
          disabled={loading}
          className="btn-destructive"
        >
          {loading ? <Spinner /> : 'Delete project'}
        </button>
      </div>
    </Modal>
  );
}
```

**Focus management:** When a modal opens, focus moves to the first focusable element inside it (or to the modal itself if no focusable element precedes the content). When it closes, focus returns to the element that triggered it.

## Related Documents

- [Design system components (full state list)](../visual/components.md)
- [Empty states](empty-states.md)
- [Error feedback](error-feedback.md)
- [Frontend accessibility](../../frontend/accessibility.md)
