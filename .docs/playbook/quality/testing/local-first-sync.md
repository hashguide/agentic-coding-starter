---
kind: playbook
domain: quality
topics: [quality, testing, local, first, sync]
authority: guidance
---

# Local-First Sync Testing

Local-first applications store data on the device first and sync to the server in the background. Testing these applications requires simulating offline conditions, reconnection, and concurrent edits from multiple clients.

## What Needs Testing

| Scenario | What can go wrong | How to test |
|---|---|---|
| **Offline → online transition** | Data entered offline doesn't sync on reconnect | Simulate network drop, make changes, restore network, verify sync |
| **Sync conflict** | Two devices edit the same record offline; one wins, one loses silently | Two clients with the same document, both go offline, both make edits, both reconnect |
| **Conflict resolution** | The wrong version wins; data is corrupted | Assert that conflict resolution produces a deterministic, expected result |
| **Partial sync** | Sync starts but network drops mid-way | Large payload, network cut after partial delivery |
| **Data integrity** | Optimistic update rolls back incorrectly | Assert that failed sync restores pre-optimistic state |

## Test Plan: Offline-to-Online Transition

```typescript
// tests/sync/offline-transition.test.ts
import { createLocalStore } from '../../src/sync/localStore';
import { createSyncEngine } from '../../src/sync/engine';

test('data entered offline syncs correctly on reconnect', async () => {
  const store = createLocalStore();
  const sync = createSyncEngine({ store, online: false }); // Start offline

  // User creates a record while offline
  await store.create({ type: 'note', content: 'Offline note' });
  expect(await store.count({ type: 'note' })).toBe(1);
  expect(await store.countPending()).toBe(1);  // 1 pending sync operation

  // User reconnects
  sync.setOnline(true);
  await sync.flush(); // Wait for pending ops to sync

  // Verify synced to server
  const serverRecord = await serverDb.note.findFirst({
    where: { content: 'Offline note' }
  });
  expect(serverRecord).toBeDefined();
  expect(await store.countPending()).toBe(0);
});
```

## Test Plan: Sync Conflict Scenario

```typescript
test('two devices editing same record offline resolve conflict deterministically', async () => {
  // Setup: Create a shared document visible to both clients
  const docId = await serverDb.document.create({ data: { content: 'Original' } });

  // Client A and B both start with the document
  const clientA = await createOfflineClient({ docId, initialContent: 'Original' });
  const clientB = await createOfflineClient({ docId, initialContent: 'Original' });

  // Both go offline
  clientA.setOnline(false);
  clientB.setOnline(false);

  // Both make conflicting edits
  await clientA.update('Client A edit');
  await clientB.update('Client B edit');

  // Client A reconnects and syncs first
  clientA.setOnline(true);
  await clientA.sync();

  // Client B reconnects and syncs (conflict!)
  clientB.setOnline(true);
  await clientB.sync();

  // Verify: one edit won, no data is silently lost
  const final = await serverDb.document.findUnique({ where: { id: docId } });
  expect(['Client A edit', 'Client B edit']).toContain(final!.content);

  // Verify: the losing edit is NOT silently dropped — it appears in conflict log or creates a conflict version
  const conflicts = await serverDb.conflictLog.findMany({ where: { documentId: docId } });
  expect(conflicts.length).toBeGreaterThan(0);
});
```

## Verification: No Silent Data Loss

The most important invariant in local-first systems is that no user action is silently discarded. Either:
- The edit wins the conflict and becomes the canonical version
- The edit loses the conflict but appears in a conflict log, undo history, or conflict UI

```typescript
test('no data is silently lost during conflict resolution', async () => {
  // After the conflict test above:
  const conflicts = await serverDb.conflictLog.count({ where: { documentId: docId } });
  const currentVersion = await serverDb.document.findUnique({ where: { id: docId } });

  // Either the conflict log has the losing edit, or the current version has it
  const allContent = [
    currentVersion!.content,
    ...conflicts.map(c => c.content)
  ];

  expect(allContent).toContain('Client A edit');
  expect(allContent).toContain('Client B edit');
});
```

## Network Simulation

Use a network mock to simulate offline conditions in tests:

```typescript
// Simulated network layer
class MockNetwork {
  private isOnline = true;
  private pendingRequests: Array<() => Promise<void>> = [];

  setOnline(value: boolean) {
    this.isOnline = value;
    if (value) {
      // Flush pending requests when coming back online
      Promise.all(this.pendingRequests.map(fn => fn()));
      this.pendingRequests = [];
    }
  }

  async fetch(url: string, options: RequestInit) {
    if (!this.isOnline) {
      throw new Error('Network unavailable');
    }
    return globalThis.fetch(url, options);
  }
}
```

## Related Documents

- [Testing strategy](strategy.md)
- [Testing integration](integration.md)
- [Design interaction optimistic-ui](../../experience/interaction/optimistic-ui.md)
