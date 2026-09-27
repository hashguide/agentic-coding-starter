---
kind: playbook
domain: guides
topics: [guides, setup, data, export, and, deletion]
authority: guidance
---

# How to Setup Data Export and Account Deletion

**What you'll have at the end:** A compliant data export flow and a legally correct account deletion flow (hard delete + Stripe cancellation + session destruction).

---

## Part 1: Data Export

### Step 1.1 — Create the Export Background Job

```typescript
// workers/exportJob.ts
import { exportQueue } from './queues';
import archiver from 'archiver';
import { createWriteStream } from 'fs';
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';

const s3 = new S3Client({ region: process.env.AWS_REGION! });

exportQueue.process('data-export', async (job) => {
  const { userId } = job.data as { userId: string };

  const user = await db.user.findUnique({
    where: { id: userId },
    include: {
      // Include all user-owned data relationships
      posts: true,
      comments: true,
      // ...other owned data
    },
  });

  if (!user) throw new Error(`User ${userId} not found`);

  // Assemble the export archive
  const exportData = {
    profile: { email: user.email, name: user.name, createdAt: user.createdAt },
    // ...include all user-owned data
  };

  const jsonBuffer = Buffer.from(JSON.stringify(exportData, null, 2), 'utf-8');
  const key = `exports/${userId}/${Date.now()}.json`;

  await s3.send(
    new PutObjectCommand({
      Bucket: process.env.S3_BUCKET!,
      Key: key,
      Body: jsonBuffer,
      ContentType: 'application/json',
    })
  );

  // Generate a time-limited signed URL (valid for 48 hours)
  const downloadUrl = await getSignedUrl(
    s3,
    new GetObjectCommand({ Bucket: process.env.S3_BUCKET!, Key: key }),
    { expiresIn: 48 * 60 * 60 }
  );

  // Email the download link
  await sendExportReadyEmail({ email: user.email, name: user.name, downloadUrl });

  // Create an in-app notification with direct link to settings
  await createNotification({
    userId,
    type: 'success',
    message: 'Your data export is ready. Download link sent to your email.',
    link: '/settings/data',
  });

  // Update the user's export status
  await db.user.update({
    where: { id: userId },
    data: { dataExportStatus: 'completed', dataExportCompletedAt: new Date() },
  });
});
```

### Step 1.2 — Create the Export API Endpoint

```typescript
// POST /api/settings/export
app.post('/api/settings/export', requireAuth, async (req, res) => {
  const userId = req.session.userId!;

  const user = await db.user.findUnique({
    where: { id: userId },
    select: { dataExportStatus: true, dataExportRequestedAt: true },
  });

  // Prevent duplicate exports (one active at a time)
  if (user?.dataExportStatus === 'pending') {
    return res.status(409).json({ error: 'An export is already in progress' });
  }

  await db.user.update({
    where: { id: userId },
    data: { dataExportStatus: 'pending', dataExportRequestedAt: new Date() },
  });

  await exportQueue.add('data-export', { userId });

  res.json({ status: 'pending' });
});

// GET /api/settings/export/status
app.get('/api/settings/export/status', requireAuth, async (req, res) => {
  const user = await db.user.findUnique({
    where: { id: req.session.userId! },
    select: { dataExportStatus: true, dataExportCompletedAt: true },
  });
  res.json({ status: user?.dataExportStatus ?? 'idle', completedAt: user?.dataExportCompletedAt });
});
```

### Step 1.3 — Build the Export UI (Settings Page)

```tsx
// In Settings → Data & Privacy section
function DataExportSection() {
  const { data: status, refetch } = useQuery({
    queryKey: ['export-status'],
    queryFn: () => fetch('/api/settings/export/status').then((r) => r.json()),
    refetchInterval: status?.status === 'pending' ? 5000 : false,
  });

  async function requestExport() {
    await fetch('/api/settings/export', { method: 'POST' });
    refetch();
  }

  return (
    <div>
      <h3>Export Your Data</h3>
      <p>Download a copy of all data associated with your account.</p>
      {status?.status === 'pending' ? (
        <p>Your export is being prepared. You will receive an email when it's ready.</p>
      ) : (
        <button onClick={requestExport}>Request Data Export</button>
      )}
    </div>
  );
}
```

---

## Part 2: Account Deletion

### Step 2.1 — Build the Danger Zone UI

The user must type their account email to confirm deletion. This prevents accidental deletions.

```tsx
// In Settings → Data & Privacy → Danger Zone
function AccountDeletionSection() {
  const [confirmEmail, setConfirmEmail] = useState('');
  const [showModal, setShowModal] = useState(false);
  const { user } = useAuth();

  async function confirmDeletion() {
    if (confirmEmail !== user.email) return;

    await fetch('/api/settings/delete-account', { method: 'POST' });
    // Session will be destroyed by the server; redirect to confirmation
    window.location.href = '/account-deleted';
  }

  return (
    <div style={{ border: '1px solid var(--color-error)', borderRadius: 'var(--radius-md)', padding: 16 }}>
      <h3>Delete Account</h3>
      <p>
        Permanently delete your account and all associated data. This action cannot be undone.
      </p>
      <button
        onClick={() => setShowModal(true)}
        style={{ background: 'var(--color-error)', color: '#fff' }}
      >
        Delete Account
      </button>

      {showModal && (
        <div role="dialog" aria-modal="true" aria-labelledby="delete-dialog-title">
          <h4 id="delete-dialog-title">Confirm Account Deletion</h4>
          <p>
            This will permanently delete your account and all your data. Any active subscriptions
            will be cancelled immediately.
          </p>
          <label>
            Type <strong>{user.email}</strong> to confirm:
            <input
              type="email"
              value={confirmEmail}
              onChange={(e) => setConfirmEmail(e.target.value)}
              placeholder={user.email}
              autoComplete="off"
            />
          </label>
          <button
            onClick={confirmDeletion}
            disabled={confirmEmail !== user.email}
            style={{ background: 'var(--color-error)', color: '#fff' }}
          >
            Permanently Delete Account
          </button>
          <button onClick={() => setShowModal(false)}>Cancel</button>
        </div>
      )}
    </div>
  );
}
```

### Step 2.2 — Implement the Deletion API Endpoint

```typescript
// POST /api/settings/delete-account
app.post('/api/settings/delete-account', requireAuth, async (req, res) => {
  const userId = req.session.userId!;

  // Enqueue the deletion job
  await deletionQueue.add('account-deletion', { userId });

  // Destroy the session immediately
  req.session.destroy((err) => {
    if (err) console.error('Session destroy error:', err);
  });

  res.clearCookie('session');
  res.json({ success: true });
});
```

### Step 2.3 — Create the Deletion Background Job

```typescript
// workers/deletionJob.ts
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);

deletionQueue.process('account-deletion', async (job) => {
  const { userId } = job.data as { userId: string };

  const user = await db.user.findUnique({
    where: { id: userId },
    select: { email: true, name: true, stripeCustomerId: true },
  });

  if (!user) return; // Already deleted

  // 1. Cancel any active Stripe subscription
  if (user.stripeCustomerId) {
    const subscriptions = await stripe.subscriptions.list({
      customer: user.stripeCustomerId,
      status: 'active',
    });
    for (const sub of subscriptions.data) {
      await stripe.subscriptions.cancel(sub.id);
    }
  }

  // 2. Hard delete or anonymize all PII in all tables
  // Option A: Full hard delete (deletes the user row and all owned data via CASCADE)
  await db.user.delete({ where: { id: userId } });

  // Option B: Anonymize (retain rows for audit/analytics but remove all PII)
  // await db.user.update({
  //   where: { id: userId },
  //   data: {
  //     email: `deleted-${userId}@deleted.invalid`,
  //     name: 'Deleted User',
  //     avatarUrl: null,
  //     phone: null,
  //     deletedAt: new Date(),
  //   },
  // });

  // 3. Send deletion confirmation email (before the record is deleted)
  await sendDeletionConfirmationEmail({ email: user.email, name: user.name });

  // Note: Session was already destroyed in the API handler
});
```

**Soft delete is not sufficient.** A `deleted_at` timestamp preserves PII in the database and does not satisfy a legal erasure request under GDPR Article 17. The deletion job must hard-delete or fully anonymize all PII fields.

---

## Step 3 — Verify

**Export:**
- [ ] Clicking "Request Data Export" sets status to "pending" in the UI
- [ ] Worker processes the export and emails a download link
- [ ] Download link works and contains all user-owned data
- [ ] In-app notification appears with link to settings
- [ ] Attempting a second export while pending returns a conflict error

**Deletion:**
- [ ] The confirm button is disabled until the email matches exactly
- [ ] Submission destroys the session and redirects to `/account-deleted`
- [ ] The background job cancels any active Stripe subscription
- [ ] The user's PII is hard-deleted or fully anonymized (verified in DB)
- [ ] A deletion confirmation email is sent

---

## Related Documents

- [Database schema design](../data/schema-design.md)
- [Security data privacy](../security/data-privacy.md)
- [Compliance GDPR](../compliance/gdpr.md)
- [Design layouts settings](../experience/layouts/settings.md)
