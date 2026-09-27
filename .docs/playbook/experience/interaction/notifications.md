---
kind: playbook
domain: experience
topics: [experience, interaction, notifications]
authority: guidance
---

# Notifications

The in-app notification system keeps users informed about relevant events without requiring them to poll manually or miss important activity. A well-designed notification system has a persistent bell icon, a structured panel, clear read/unread states, and bulk actions. Notifications survive page reloads — they are stored in the database, not memory.

## Bell Icon and Badge

The bell icon lives in the top-right of the primary navigation bar. It shows an unread count badge when there are unread notifications. The badge disappears entirely when all notifications are read — it does not display a zero.

```tsx
function NotificationBell({ unreadCount }: { unreadCount: number }) {
  return (
    <button aria-label={`Notifications${unreadCount > 0 ? `, ${unreadCount} unread` : ''}`}>
      <BellIcon />
      {unreadCount > 0 && (
        <span className="badge" aria-hidden="true">
          {unreadCount > 99 ? '99+' : unreadCount}
        </span>
      )}
    </button>
  );
}
```

The badge is visually distinct: a solid filled circle in the error/accent color, positioned at the top-right corner of the bell icon.

## Notification Panel

The panel is a flyout drawer anchored to the bell icon. It opens on bell click and closes on outside click or Escape keypress.

```tsx
function NotificationPanel({ open, onClose }: { open: boolean; onClose: () => void }) {
  const { data: notifications, refetch } = useNotifications();

  async function markAllRead() {
    await fetch('/api/notifications/read-all', { method: 'POST' });
    refetch();
  }

  if (!open) return null;

  return (
    <>
      <div className="panel-backdrop" onClick={onClose} aria-hidden="true" />
      <div className="notification-panel" role="dialog" aria-label="Notifications">
        <div className="panel-header">
          <h2>Notifications</h2>
          {notifications?.some((n) => !n.readAt) && (
            <button onClick={markAllRead}>Mark all as read</button>
          )}
        </div>

        {notifications?.length === 0 ? (
          <NotificationsEmptyState />
        ) : (
          <ul>
            {notifications?.map((n) => (
              <NotificationItem key={n.id} notification={n} onDismiss={refetch} />
            ))}
          </ul>
        )}
      </div>
    </>
  );
}
```

## Notification Item Anatomy

Each notification contains:
- **Type icon:** The category of event (comment, payment, system alert) — never rely on color alone
- **Message text:** One sentence written to micro-content standards — specific, action-oriented
- **Relative timestamp:** "2 minutes ago", "Yesterday" — not an absolute datetime for recent notifications
- **Direct link:** To the exact record or action this notification concerns — never to the home dashboard
- **Dismiss button:** Visible on hover — removes the notification from the list

```tsx
function NotificationItem({ notification, onDismiss }: NotificationItemProps) {
  async function dismiss() {
    await fetch(`/api/notifications/${notification.id}`, { method: 'DELETE' });
    onDismiss();
  }

  async function markRead() {
    if (!notification.readAt) {
      await fetch(`/api/notifications/${notification.id}/read`, { method: 'POST' });
    }
  }

  return (
    <li className={cn('notification-item', !notification.readAt && 'notification-unread')}>
      <NotificationTypeIcon type={notification.type} aria-hidden="true" />
      <div className="notification-body">
        <a href={notification.linkUrl} onClick={markRead} className="notification-message">
          {notification.message}
        </a>
        <time dateTime={notification.createdAt.toISOString()}>
          {formatRelativeTime(notification.createdAt)}
        </time>
      </div>
      <button
        onClick={dismiss}
        aria-label="Dismiss notification"
        className="notification-dismiss"
      >
        ×
      </button>
    </li>
  );
}
```

## Read / Unread Visual Differentiation

| State | Visual treatment |
|---|---|
| **Unread** | Higher visual weight: bold message text, subtle highlighted background or left-border accent in the primary color |
| **Read** | Normal weight text, muted background — the default state |

The visual difference must be apparent without relying solely on color. Bold text weight is sufficient as the primary differentiator.

## Server-Side Persistence

Notifications are stored in the database. They survive page reloads, device switches, and browser sessions.

```sql
-- Prisma schema
model Notification {
  id        String    @id @default(uuid())
  userId    String
  type      String    -- 'comment_added' | 'payment_failed' | 'export_ready' etc.
  message   String
  linkUrl   String
  readAt    DateTime?
  createdAt DateTime  @default(now())
  user      User      @relation(fields: [userId], references: [id])

  @@index([userId, readAt])
}
```

```typescript
// API routes
app.get('/api/notifications', requireAuth, asyncHandler(async (req, res) => {
  const notifications = await db.notification.findMany({
    where: { userId: req.userId },
    orderBy: { createdAt: 'desc' },
    take: 50,
  });
  res.json(notifications);
}));

app.post('/api/notifications/read-all', requireAuth, asyncHandler(async (req, res) => {
  await db.notification.updateMany({
    where: { userId: req.userId, readAt: null },
    data: { readAt: new Date() },
  });
  res.json({ ok: true });
}));

app.delete('/api/notifications/:id', requireAuth, asyncHandler(async (req, res) => {
  await db.notification.deleteMany({
    where: { id: req.params.id, userId: req.userId },
  });
  res.json({ ok: true });
}));
```

## Notification Linking Rule

Every notification must link directly and specifically to the record or action it concerns. A notification saying "A comment was added to your project" must link to that specific comment — not to the project list, not to the dashboard home.

The user clicked the notification because they want to see the thing. Don't make them hunt for it.

## Related Documents

- [Empty states](empty-states.md)
- [Settings layout (notification preferences)](../layouts/settings.md)
- [Email notifications](../../integrations/email/notifications.md)
- [Notification system setup how-to](../../guides/setup-notifications.md)
