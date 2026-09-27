---
kind: playbook
domain: guides
topics: [guides, setup, notifications]
authority: guidance
---

# How to Setup In-App Notifications

**What you'll have at the end:** A complete in-app notification system — bell icon with unread badge, notification panel with read/unread states, mark-all-read, per-item delete, and server-side persistence.

---

## Step 1 — Create the Notifications Table

```prisma
// prisma/schema.prisma
model Notification {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  type      String                         // "info" | "success" | "warning" | "error"
  message   String
  link      String?                        // Direct link to the relevant resource
  read      Boolean  @default(false)
  createdAt DateTime @default(now())

  @@index([userId, read])
  @@index([userId, createdAt])
}
```

```bash
npx prisma migrate dev --name add-notifications
```

---

## Step 2 — Build API Endpoints

```typescript
// GET /api/notifications
app.get('/api/notifications', requireAuth, async (req, res) => {
  const notifications = await db.notification.findMany({
    where: { userId: req.session.userId! },
    orderBy: { createdAt: 'desc' },
    take: 50,
  });
  res.json(notifications);
});

// PATCH /api/notifications/read-all
app.patch('/api/notifications/read-all', requireAuth, async (req, res) => {
  await db.notification.updateMany({
    where: { userId: req.session.userId!, read: false },
    data: { read: true },
  });
  res.json({ success: true });
});

// PATCH /api/notifications/:id/read
app.patch('/api/notifications/:id/read', requireAuth, async (req, res) => {
  await db.notification.updateMany({
    where: { id: req.params.id, userId: req.session.userId! },
    data: { read: true },
  });
  res.json({ success: true });
});

// DELETE /api/notifications/:id
app.delete('/api/notifications/:id', requireAuth, async (req, res) => {
  await db.notification.deleteMany({
    where: { id: req.params.id, userId: req.session.userId! },
  });
  res.json({ success: true });
});
```

---

## Step 3 — Build a Utility to Create Notifications

```typescript
// lib/notifications.ts
export async function createNotification(params: {
  userId: string;
  type: 'info' | 'success' | 'warning' | 'error';
  message: string;
  link?: string;
}) {
  return db.notification.create({ data: params });
}

// Usage example: send a notification when export is ready
await createNotification({
  userId: user.id,
  type: 'success',
  message: 'Your data export is ready.',
  link: '/settings/data',
});
```

---

## Step 4 — Build the Bell Icon Component

```tsx
// components/NotificationBell.tsx
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';

interface Notification {
  id: string;
  type: string;
  message: string;
  link?: string;
  read: boolean;
  createdAt: string;
}

export function NotificationBell() {
  const [open, setOpen] = useState(false);

  const { data: notifications = [], refetch } = useQuery<Notification[]>({
    queryKey: ['notifications'],
    queryFn: () => fetch('/api/notifications').then((r) => r.json()),
    refetchInterval: 30_000, // Poll every 30 seconds
  });

  const unreadCount = notifications.filter((n) => !n.read).length;

  async function markAllRead() {
    await fetch('/api/notifications/read-all', { method: 'PATCH' });
    refetch();
  }

  async function deleteNotification(id: string) {
    await fetch(`/api/notifications/${id}`, { method: 'DELETE' });
    refetch();
  }

  return (
    <div style={{ position: 'relative' }}>
      <button
        aria-label={`Notifications${unreadCount > 0 ? `, ${unreadCount} unread` : ''}`}
        aria-expanded={open}
        onClick={() => setOpen((o) => !o)}
      >
        {/* Bell icon */}
        🔔
        {/* Badge — only show when there are unread notifications */}
        {unreadCount > 0 && (
          <span
            aria-hidden="true"
            style={{
              position: 'absolute',
              top: -4,
              right: -4,
              background: 'var(--color-error)',
              color: '#fff',
              borderRadius: '50%',
              fontSize: '11px',
              minWidth: '16px',
              height: '16px',
              display: 'flex',
              alignItems: 'center',
              justifyContent: 'center',
              padding: '0 3px',
            }}
          >
            {unreadCount > 99 ? '99+' : unreadCount}
          </span>
        )}
      </button>

      {open && (
        <NotificationPanel
          notifications={notifications}
          onMarkAllRead={markAllRead}
          onDelete={deleteNotification}
          onClose={() => setOpen(false)}
        />
      )}
    </div>
  );
}
```

---

## Step 5 — Build the Notification Panel

```tsx
// components/NotificationPanel.tsx
import { useEffect, useRef } from 'react';
import { formatDistanceToNow } from 'date-fns';

interface Props {
  notifications: Notification[];
  onMarkAllRead: () => void;
  onDelete: (id: string) => void;
  onClose: () => void;
}

const TYPE_ICONS: Record<string, string> = {
  info: 'ℹ️',
  success: '✅',
  warning: '⚠️',
  error: '❌',
};

export function NotificationPanel({ notifications, onMarkAllRead, onDelete, onClose }: Props) {
  const panelRef = useRef<HTMLDivElement>(null);

  // Close on outside click
  useEffect(() => {
    function handleClickOutside(e: MouseEvent) {
      if (panelRef.current && !panelRef.current.contains(e.target as Node)) {
        onClose();
      }
    }
    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, [onClose]);

  const hasUnread = notifications.some((n) => !n.read);

  return (
    <div
      ref={panelRef}
      role="dialog"
      aria-label="Notifications"
      style={{
        position: 'absolute',
        right: 0,
        top: '100%',
        width: 360,
        maxHeight: 480,
        overflowY: 'auto',
        background: 'var(--color-surface)',
        border: '1px solid var(--color-border)',
        borderRadius: 'var(--radius-lg)',
        boxShadow: 'var(--shadow-lg)',
        zIndex: 100,
      }}
    >
      {/* Header */}
      <div style={{ display: 'flex', justifyContent: 'space-between', padding: '12px 16px', borderBottom: '1px solid var(--color-border)' }}>
        <span style={{ fontWeight: 600 }}>Notifications</span>
        {hasUnread && (
          <button onClick={onMarkAllRead} style={{ fontSize: '13px' }}>
            Mark all as read
          </button>
        )}
      </div>

      {/* Notification list */}
      {notifications.length === 0 ? (
        <div style={{ padding: '24px 16px', textAlign: 'center', color: 'var(--color-text-muted)' }}>
          No notifications
        </div>
      ) : (
        <ul style={{ listStyle: 'none', margin: 0, padding: 0 }}>
          {notifications.map((n) => (
            <li
              key={n.id}
              style={{
                display: 'flex',
                gap: 10,
                padding: '12px 16px',
                borderBottom: '1px solid var(--color-border)',
                background: n.read ? 'transparent' : 'var(--color-info-bg)',
                fontWeight: n.read ? 'normal' : 600,
              }}
            >
              <span aria-hidden="true">{TYPE_ICONS[n.type] ?? 'ℹ️'}</span>
              <div style={{ flex: 1, minWidth: 0 }}>
                {n.link ? (
                  <a href={n.link} style={{ textDecoration: 'none', color: 'inherit' }}>
                    {n.message}
                  </a>
                ) : (
                  <span>{n.message}</span>
                )}
                <div style={{ fontSize: '12px', color: 'var(--color-text-muted)', marginTop: 2 }}>
                  {formatDistanceToNow(new Date(n.createdAt), { addSuffix: true })}
                </div>
              </div>
              <button
                aria-label="Dismiss notification"
                onClick={() => onDelete(n.id)}
                style={{ flexShrink: 0, alignSelf: 'flex-start' }}
              >
                ✕
              </button>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

---

## Step 6 — Place the Bell in the Navbar

```tsx
// components/Navbar.tsx
import { NotificationBell } from './NotificationBell';

export function Navbar() {
  return (
    <nav>
      {/* ...other nav items... */}
      <div style={{ marginLeft: 'auto', display: 'flex', alignItems: 'center', gap: 16 }}>
        <NotificationBell />
        {/* user avatar / menu */}
      </div>
    </nav>
  );
}
```

---

## Step 7 — Verify

- [ ] Bell icon appears in navbar
- [ ] Unread badge shows correct count and disappears when all are read (badge does not show "0")
- [ ] Clicking the bell opens the panel
- [ ] Clicking outside the panel closes it
- [ ] Unread notifications have bold text and highlighted background
- [ ] "Mark all as read" clears unread state
- [ ] Per-item delete removes the notification
- [ ] Empty state shows "No notifications"
- [ ] Notifications persist across page reload (verified from the database, not just React state)
- [ ] Direct link in a notification goes to the relevant resource, not the home dashboard

---

## Related Documents

- [Design notifications](../experience/interaction/notifications.md)
- [Email notifications](../integrations/email/notifications.md)
