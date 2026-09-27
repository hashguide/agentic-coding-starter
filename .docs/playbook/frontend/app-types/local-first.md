---
kind: playbook
domain: frontend
topics: [frontend, app, types, local, first]
authority: guidance
---

# Local-First Architecture

Local-first is an architecture where the local device is the primary data store and all user interactions operate on local data first. Changes sync to the server in the background. The result is instant perceived performance (no network round-trips for reads or writes) and full offline capability (the app works regardless of connectivity).

## The Local-First Model

```
┌──────────────────────────────────────────────────────────┐
│ Browser                                                   │
│  ┌─────────────────────────────────────────────────────┐ │
│  │ Application UI                                       │ │
│  │   Reads from / Writes to                            │ │
│  │            ↓↑                                       │ │
│  │ Local Database (IndexedDB / OPFS)                   │ │
│  │            ↓↑                                       │ │
│  │ Sync Engine (background, conflict resolution)        │ │
│  └─────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────┘
                           ↕ (background sync)
┌──────────────────────────────────────────────────────────┐
│ Server                                                    │
│  Database (source of truth for sync)                     │
└──────────────────────────────────────────────────────────┘
```

## Local Storage Options

**IndexedDB:** Browser-native key-value/object store. High capacity (~1GB+), persistent, queryable. The standard choice for local-first data.

**OPFS (Origin Private File System):** Provides synchronous file access from web workers, useful for SQLite-in-browser (via WASM). Faster than IndexedDB for bulk reads/writes.

## Sync Engine Options

| Engine | Model | Good for |
|---|---|---|
| **ElectricSQL** | PostgreSQL → client sync via Electric | Teams that want Postgres as the source of truth |
| **PowerSync** | Postgres/MongoDB → client sync | Flexible schema sync, offline-first mobile/web |
| **Replicache** | Custom sync protocol | Maximum control over sync behavior |
| **TinyBase** | Local reactive store | Simpler apps, less complex sync requirements |

## When Local-First Is the Right Architecture

**Justified when:**
- The app's core value requires offline functionality (field tools, note-taking, collaboration tools)
- Perceived performance must be instantaneous — users cannot tolerate network latency on writes
- The app is used in low/no connectivity environments

**Not justified when:**
- The app handles sensitive real-time data where server-side validation before display is required (financial transactions, medical records)
- The product is simple enough that optimistic UI (see [optimistic-ui.md](../../experience/interaction/optimistic-ui.md)) provides adequate perceived performance
- The team doesn't have the engineering capacity to implement and maintain conflict resolution logic

## Conflict Resolution Strategy

When two devices edit the same record offline, the sync merge must have a defined winner:

- **Last-write-wins:** Simpler but loses intermediate changes
- **Merge by field:** Each field is independently merged; the most-recent value per field wins
- **CRDT (Conflict-free Replicated Data Types):** Mathematically guaranteed to converge; complex to implement
- **User-prompted resolution:** Show the conflict to the user and ask them to choose (only viable for low-frequency conflicts)

Decide the conflict resolution strategy before building the sync layer. Changing it retroactively is expensive.

## Security

**Local databases must be encrypted at rest.** Local data persists after logout and can be accessed by malicious extensions or malware. Use a library that encrypts IndexedDB contents (e.g., SQLCipher for OPFS-backed SQLite).

**The server must re-validate all synced writes as if they came from an untrusted client.** A local-first client can be modified to submit arbitrary data during sync. The sync endpoint cannot trust that a record's content is valid just because it was created locally. Apply the same input validation, authorization checks, and business rule enforcement as any other API endpoint.

**Conflict resolution must not allow privilege escalation.** A merge that accepts arbitrary field values from a client could allow a client to modify fields it doesn't own (e.g., setting `userId` to a different user's ID). The server must validate the final merged record against the authenticated user's permissions.

## Related Documents

- [Client storage](client-storage.md)
- [PWA app type](pwa.md)
- [Testing local-first sync](../../quality/testing/local-first-sync.md)
