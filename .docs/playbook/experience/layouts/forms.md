---
kind: playbook
domain: experience
topics: [experience, layouts, forms]
authority: guidance
---

# Form Layout

Forms are where users accomplish things — signing up, creating records, changing settings, making purchases. Every friction point in a form is a place the user might abandon. The goal is to ask only what is needed, in the right order, with clear feedback.

## Single-Column vs. Multi-Column Layout

**Single-column layout generally outperforms multi-column for completion rate.**

A single column requires no scanning overhead. The user reads and fills linearly, top to bottom. Multi-column forms require the eye to jump across the form to determine order of completion, which increases cognitive load.

**Use single-column for:**
- Signup, login, and checkout forms
- Any form where completion rate is the primary metric
- Mobile-first forms

**Use multi-column for:**
- Forms with closely-related side-by-side fields where the relationship is visually meaningful (First name / Last name, Country / State / Zip, Start date / End date)
- Admin forms where density is a higher priority than conversion (editing an internal record)
- Always test multi-column against single-column if conversion is important

## Logical Field Ordering

Field order should match the user's mental model of the task, not the order of columns in the database:

**Signup:** Email → Password → (optional: name) → CTA. Do not ask for a name before email in a signup flow — email is the identifier, and the user expects it first.

**Profile completion:** Full name → Display name → Bio → Location — personal fields first, public representation fields second.

**Billing:** Card details → Billing address → Review + submit. The card is the primary thing; address is supplementary context the payment processor needs.

**Settings updates:** The field to change → Confirmation (if destructive) → Save. Do not spread dependent fields apart.

## Inline Validation

Validation errors are anchored below the specific field that caused the error. They appear on blur (when the user leaves the field), not on every keystroke.

```tsx
function Field({ label, id, error, children }: FieldProps) {
  return (
    <div className="field">
      <label htmlFor={id}>{label}</label>
      {children}
      {error && (
        <span id={`${id}-error`} role="alert" className="field-error">
          {error}
        </span>
      )}
    </div>
  );
}

// Input with error association
<Field label="Email" id="email" error={errors.email}>
  <input
    id="email"
    type="email"
    aria-describedby={errors.email ? 'email-error' : undefined}
    aria-invalid={!!errors.email}
    onBlur={() => validateField('email')}
  />
</Field>
```

**Error message rules:**
- Say what is wrong and how to fix it: "Must be a valid email address", not "Invalid input"
- Do not say "Email is required" — it says nothing useful. Say "Enter your email address to continue."
- Never use field-level errors to convey server-level security information (see Security note below)

## Submit Button Behavior

The submit button must:
- Enter a loading state on submit (spinner replaces or accompanies label)
- Remain disabled until the request resolves (prevents double-submit)
- Return to default state on error (so the user can retry)

## Security

**Form errors must never reveal server-side state.** A field error that says "that email is already registered" tells an attacker which email addresses are in the database. The server returns an `AUTH_FAILED` or `CONFLICT` code; the client maps it to a safe display string.

For email fields on signup: if the email is already registered, the server sends a "you already have an account" email to that address and returns a response identical to a successful signup (HTTP 200, same body). The UI shows a "check your email" message. The user never knows whether the email was new or existing. See [backend/error-messages.md](../../backend/runtime/error-messages.md) for the server-side implementation.

## Related Documents

- [Error feedback](../interaction/error-feedback.md)
- [Backend error messages](../../backend/runtime/error-messages.md)
- [Auth pages](auth-pages.md)
- [Component states (form states)](../interaction/component-states.md)
