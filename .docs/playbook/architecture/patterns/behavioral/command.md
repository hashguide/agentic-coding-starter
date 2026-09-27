---
kind: playbook
domain: architecture
topics: [architecture, patterns, behavioral, command]
authority: guidance
---

# Command Pattern

The Command pattern encapsulates an action as an object. Instead of calling a function directly, you create a command object that knows how to execute the action, along with any data it needs. This enables queuing commands for later execution, logging all executed commands, and implementing undo/redo.

## Definition

A command is an object with an `execute()` method (and optionally an `undo()` method) that encapsulates both the action and all its parameters. A queue, scheduler, or invoker stores and calls commands without knowing their specifics.

## Code Example

Encapsulating a background job as a command object:

```typescript
interface Command<TResult = void> {
  execute(): Promise<TResult>;
}

// Command encapsulating account deletion
class DeleteAccountCommand implements Command {
  constructor(
    private readonly params: {
      userId: string;
      requestedAt: Date;
    },
  ) {}

  async execute(): Promise<void> {
    const { userId } = this.params;

    // Hard-delete or anonymize all PII (not soft delete)
    await db.$transaction(async (tx) => {
      await tx.user.update({
        where: { id: userId },
        data: {
          email: `deleted-${userId}@deleted.invalid`,
          name: '[Deleted User]',
          deletedAt: new Date(),
        },
      });
      await tx.session.deleteMany({ where: { userId } });
      await tx.notification.deleteMany({ where: { userId } });
    });

    // Cancel Stripe subscription
    const subscription = await db.subscription.findUnique({ where: { userId } });
    if (subscription?.stripeSubscriptionId) {
      await stripe.subscriptions.cancel(subscription.stripeSubscriptionId);
    }

    // Send confirmation email
    await emailService.sendTransactional({
      to: '[from the pre-deletion record]',
      subject: 'Your account has been deleted',
      html: deletionConfirmationTemplate(),
      text: 'Your account and all associated data have been permanently deleted.',
    });
  }
}

// Enqueue the command as a serializable job
await jobQueue.add('deleteAccount', {
  userId,
  requestedAt: new Date(),
});

// In the worker, deserialize and execute
jobQueue.process('deleteAccount', async (job) => {
  const command = new DeleteAccountCommand(job.data);
  await command.execute();
});
```

## Relevance to Undo/Redo

For features with undo functionality, the command pattern enables it cleanly:

```typescript
interface UndoableCommand<T = void> {
  execute(): Promise<T>;
  undo(): Promise<void>;
}

class MoveCardCommand implements UndoableCommand {
  private previousColumnId: string | null = null;

  constructor(private readonly params: { cardId: string; targetColumnId: string }) {}

  async execute(): Promise<void> {
    const card = await db.card.findUniqueOrThrow({ where: { id: this.params.cardId } });
    this.previousColumnId = card.columnId;
    await db.card.update({
      where: { id: this.params.cardId },
      data: { columnId: this.params.targetColumnId },
    });
  }

  async undo(): Promise<void> {
    if (!this.previousColumnId) return;
    await db.card.update({
      where: { id: this.params.cardId },
      data: { columnId: this.previousColumnId },
    });
  }
}
```

## Relevance to Job Queue Design

Background jobs are naturally a command pattern: the job payload is the command's parameters, the worker's `process` function is the invoker, and each job type is a concrete command class. See [workers/job-patterns.md](../../../backend/async/job-patterns.md) for the job queue implementation.

## Related Documents

- [Workers job patterns](../../../backend/async/job-patterns.md)
- [Observer pattern](observer.md)
- [Behavioral patterns overview](overview.md)
