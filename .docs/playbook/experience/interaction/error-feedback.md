---
kind: playbook
domain: experience
topics: [experience, interaction, error, feedback]
authority: guidance
---

# Error Feedback

Error display is one of the highest-leverage UI decisions. A well-placed, clearly worded error helps the user fix a problem in seconds. A raw server message, a modal for a trivial input error, or an ambiguous "Something went wrong" wastes time and erodes trust.

## Error Display Decision Rule

| Error type | Display mechanism |
|---|---|
| Background / async failure (network error, webhook failed, job failed) | Auto-dismissing toast notification — appears without interrupting the user's current focus |
| Synchronous form validation failure | Anchored message below the specific field that failed — never a toast for field errors |
| Optimistic UI rollback | Inline error near the affected element — where the action was taken |
| Destructive action failure | Inline within the modal or confirmation dialog where the action originated |

**Principle:** Errors that interrupt flow should be proportional to the severity. A required field isn't an interruption event. An async system failure is.

## Toast Component Anatomy

An auto-dismissing toast notification requires all of these elements:

- **Icon:** Visual type indicator (error, warning, info, success) — color is never the only signal
- **Short message:** One or two sentences. "Couldn't save your changes. Try again." Not a paragraph.
- **Optional action button:** "Retry" or "View details" — inline, not a separate dialog
- **Auto-dismiss timer:** 4–6 seconds for informational; 8–10 seconds for errors (errors need more time to read and act on)
- **Pause on hover:** The timer pauses when the user hovers over the toast — they may be reading it
- **Manual close button:** Always present — the user should always be able to dismiss immediately

```tsx
function Toast({ type, message, action, onDismiss }: ToastProps) {
  const timerRef = React.useRef<ReturnType<typeof setTimeout>>();
  const [paused, setPaused] = React.useState(false);
  const duration = type === 'error' ? 8000 : 5000;

  React.useEffect(() => {
    if (!paused) {
      timerRef.current = setTimeout(onDismiss, duration);
    }
    return () => clearTimeout(timerRef.current);
  }, [paused, onDismiss, duration]);

  return (
    <div
      role="alert"
      aria-live="assertive"
      className={`toast toast-${type}`}
      onMouseEnter={() => setPaused(true)}
      onMouseLeave={() => setPaused(false)}
    >
      <ToastIcon type={type} aria-hidden="true" />
      <span>{message}</span>
      {action && (
        <button onClick={action.handler} className="toast-action">
          {action.label}
        </button>
      )}
      <button onClick={onDismiss} aria-label="Dismiss" className="toast-close">×</button>
    </div>
  );
}
```

## Error Code Mapping Table

The server returns machine-readable error codes. The client maps them to safe, user-friendly display strings. The client never forwards raw server messages.

| Server error code | User-facing display string |
|---|---|
| `AUTH_FAILED` | "Incorrect email or password." |
| `RATE_LIMITED` | "Too many attempts. Please try again later." |
| `VALIDATION_ERROR` | Field-specific message from the error payload |
| `NOT_FOUND` | "We couldn't find what you're looking for." |
| `FORBIDDEN` | "You don't have permission to do that." |
| `SUBSCRIPTION_REQUIRED` | "This feature requires an active subscription." |
| `PAYMENT_FAILED` | "Your payment couldn't be processed. Please update your payment method." |
| `NETWORK_ERROR` (client-side) | "Couldn't reach the server. Check your connection and try again." |
| Any unhandled server error | "Something unexpected happened. Try again, or contact support if this continues." |

## Auth Enumeration Rule (UI Layer)

A failed login attempt must always display "Incorrect email or password" — regardless of whether the email address doesn't exist or the password is wrong. The UI must not distinguish between these two cases.

**Why:** Displaying "That email isn't registered" tells an attacker which emails exist in the system. An attacker can enumerate email addresses at scale by submitting them against the login form and reading the error message. Identical errors for both cases eliminate this vector.

The server enforces this by always returning `AUTH_FAILED` for both cases (see [backend/error-messages.md](../../backend/runtime/error-messages.md)). The client completes the control by mapping `AUTH_FAILED` to one display string regardless of which field the user filled in.

## Form Field Error Placement

Field errors are anchored below the specific field that caused the error, not in a summary at the top of the form:

```tsx
<div className="field">
  <label htmlFor="email">Email</label>
  <input
    id="email"
    type="email"
    aria-describedby={error ? 'email-error' : undefined}
    aria-invalid={!!error}
  />
  {error && (
    <span id="email-error" role="alert" className="field-error">
      {error}
    </span>
  )}
</div>
```

Field errors appear on blur (when the user leaves the field), not on every keystroke. Triggering on every keystroke punishes the user for being mid-sentence.

## Security

**The client must never display raw server error responses.** Stack traces, database error messages, SQL queries, internal field names, and server-side state must never reach the user. The server returns an error code; the client decides what to show. This is not optional — raw server errors reveal implementation details (table names, ORM names, internal logic) that directly assist attackers.

**Auth errors are always identical at the UI layer.** The auth enumeration rule is both a UX rule (consistent messaging) and a security control. A UI that deviates from it — even for "helpfulness" — weakens the server-side protection.

## Related Documents

- [Backend error messages (server-side rule)](../../backend/runtime/error-messages.md)
- [Auth pages layout (UI rule)](../layouts/auth-pages.md)
- [Testing error scenarios (test assertions)](../../quality/testing/error-scenarios.md)
- [Empty states](empty-states.md)
