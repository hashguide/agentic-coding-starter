---
kind: playbook
domain: experience
topics: [experience, layouts, settings]
authority: guidance
---

# Settings Page

The settings page is the self-service control center for the user's account. A well-structured settings page reduces support requests, gives users autonomy, and handles the full account lifecycle — including data export and account deletion.

## Information Architecture

Settings are grouped into sections, navigated via a sidebar (desktop) or tab navigation (mobile). At minimum, include these sections:

| Section | Content |
|---|---|
| **Profile** | Display name, avatar, bio, contact information |
| **Account** | Email address, password change, username |
| **Notifications** | Per-type notification preferences (in-app and email, independently) |
| **Billing** | Current plan, payment method, invoice history, upgrade/downgrade |
| **Security** | Active sessions, two-factor authentication, API keys |
| **Data & Privacy** | Data export, account deletion |

```
┌──────────────────┬────────────────────────────────────┐
│ Profile          │ [Settings content for active item] │
│ Account          │                                     │
│ Notifications    │                                     │
│ Billing          │                                     │
│ Security         │                                     │
│ Data & Privacy   │                                     │
└──────────────────┴────────────────────────────────────┘
```

## Inline Edit vs. Modal Edit

**Inline edit:** The field is displayed as read-only text. Clicking an "Edit" pencil icon converts it to an editable input in place. Saving updates the field without a page reload.

**Best for:** Single-field changes where modal overhead is disproportionate (display name, bio, contact email).

**Modal edit:** A dialog opens with the form to change a value. Used when:
- The change requires a confirmation step (email change → verify new email)
- Multiple related fields are changed together (billing address)
- The change has security implications (password change, 2FA setup)

## Notifications Section

The notifications section allows the user to configure which types of notifications they receive, independently for in-app and email channels:

```
Notifications

Type                    In-app    Email
──────────────────────────────────────────
New comment             [✓]       [✓]
Export ready            [✓]       [✓]
Team member joined      [✓]       [ ]
Billing updates         [✓]       [✓]
Security alerts         [✓]       [✓]  ← always on; cannot be disabled
```

Security alerts (login from new device, password changed) should not be disableable — they are a safety mechanism, not a preference.

## Data & Privacy Section

### Download My Data

Triggers an async background job that assembles all user-owned data into a downloadable archive.

**UI states:**

1. **Default:** "Download my data" button with description: "Export all your data as a JSON archive."
2. **In progress:** Button replaced by: "Preparing your export... You'll receive an email when it's ready." (Non-dismissible; does not block the page)
3. **Completed:** Success notification with a download link (also sent by email). The download link is time-limited (48 hours).

```tsx
function DataExportSection() {
  const [status, setStatus] = React.useState<'idle' | 'pending' | 'done'>('idle');

  async function requestExport() {
    setStatus('pending');
    await fetch('/api/account/export', { method: 'POST' });
    // The email will be sent when the job completes
  }

  if (status === 'pending') {
    return <p>Preparing your export... You'll receive an email when it's ready.</p>;
  }

  return (
    <div>
      <p>Export all your data as a downloadable archive.</p>
      <button onClick={requestExport}>Download my data</button>
    </div>
  );
}
```

### Delete My Account

Visually separated from the rest of the settings page in a "Danger zone" section with a warning border.

**Requirements:**
- Explain clearly what will be permanently deleted and that the action cannot be undone
- Require the user to type their account email address (or a specific confirmation phrase like "delete my account") before the button becomes enabled
- On confirmation, display a final confirmation modal before processing

**What happens on confirmation:**
- A background job is enqueued that hard-deletes or fully anonymizes all PII — this is not a soft delete
- Active Stripe subscription is cancelled
- A deletion confirmation email is sent to the user's email address
- The user's active session is destroyed

**This is not a soft delete.** A user initiating account deletion is exercising their right to erasure. Soft-deleting the account (setting `deleted_at`) leaves all their PII in the database. See [database/schema-design.md](../../data/schema-design.md) and [security/data-privacy.md](../../security/data-privacy.md).

```tsx
function DeleteAccountSection() {
  const [confirmEmail, setConfirmEmail] = React.useState('');
  const [modalOpen, setModalOpen] = React.useState(false);
  const { user } = useCurrentUser();
  const canDelete = confirmEmail === user.email;

  return (
    <div className="danger-zone">
      <h3>Delete account</h3>
      <p>
        Permanently deletes your account, all your data, and cancels your subscription.
        <strong>This cannot be undone.</strong>
      </p>

      <label>
        Type <code>{user.email}</code> to confirm:
        <input
          type="email"
          value={confirmEmail}
          onChange={(e) => setConfirmEmail(e.target.value)}
          autoComplete="off"
        />
      </label>

      <button
        onClick={() => setModalOpen(true)}
        disabled={!canDelete}
        className="btn-destructive"
      >
        Delete my account
      </button>

      <DeleteAccountModal
        open={modalOpen}
        onClose={() => setModalOpen(false)}
        onConfirm={async () => {
          await fetch('/api/account/delete', { method: 'POST' });
          // Session destroyed server-side; redirect to goodbye page
          window.location.href = '/goodbye';
        }}
      />
    </div>
  );
}
```

## Security

**Account deletion must hard-delete or fully anonymize all PII.** A soft delete that sets `deleted_at` is not acceptable for a user-initiated erasure request. The user's data must be permanently removed or rendered unidentifiable. Setting `deleted_at` leaves the email address, name, payment history, and all other PII intact in the database — that is not deletion.

## Related Documents

- [Security data privacy](../../security/data-privacy.md)
- [Database schema design](../../data/schema-design.md)
- [Notification system](../interaction/notifications.md)
- [Email notifications](../../integrations/email/notifications.md)
- [Data export and deletion how-to](../../guides/setup-data-export-and-deletion.md)
