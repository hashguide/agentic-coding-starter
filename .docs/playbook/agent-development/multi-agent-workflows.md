---
kind: playbook
domain: agent-development
topics: [agent, development, multi, workflows]
authority: guidance
---

# Multi-Agent Workflows

Running multiple AI agent sessions concurrently on a single codebase creates coordination risks that don't exist in single-session work. Two sessions editing the same file will produce a merge conflict at best and silently overwrite each other's work at worst. This document covers how to divide work to make parallel agent sessions productive rather than chaotic.

## Dividing Work by Domain/Directory

The safest division is by directory or domain — one session owns one domain, touches only the files in that domain, and does not reach into another session's territory.

**Example division:**

| Session | Domain | Files |
|---|---|---|
| Session A | Auth | `src/auth/`, `src/middleware/auth.ts` |
| Session B | Notifications | `src/notifications/`, `src/components/NotificationBell.tsx` |
| Session C | Billing | `src/billing/`, `src/integrations/stripe/` |

Sessions A, B, and C can proceed concurrently without conflicts because they own non-overlapping file sets.

**Shared files are a risk:** If multiple sessions touch `src/lib/db.ts`, `src/server.ts`, or `schema.prisma`, they will conflict. Assign shared infrastructure files to one session (or handle them manually) before distributing domain work.

## The Risk of Concurrent Sessions on the Same Files

Two sessions editing `schema.prisma` simultaneously will produce:
- Conflicting migration files with the same timestamp prefix
- A merge conflict in the schema file
- Possibly invalid migration history if one session applied its migration before the merge

Two sessions editing `src/server.ts` simultaneously will produce conflicting route registrations that are easy to miss during review.

**Rule:** One active session per logical unit of work. If two domains require changes to the same shared file, serialize that work: finish one, merge it, then start the other.

## Recommended Pattern

1. **Plan before spawning.** Identify which domains need work and which shared files each domain touches. Resolve overlaps before starting any session.
2. **One active session per logical unit.** Do not start Session B until Session A's work is merged.
3. **Merge before starting the next session.** A session always starts from the current state of main. Diverged branches compound conflicts.
4. **Exception: independent, non-overlapping work.** If Session A touches only `src/notifications/` and Session B touches only `src/billing/`, and neither touches shared infrastructure, they can proceed in parallel safely.

## Communication Across Sessions

Sessions do not share memory. If Session A establishes a pattern (e.g., "all API errors return a structured object with `code` and `message` fields"), Session B does not know this unless you tell it. Start each session with a decision summary (see [context-management.md](context-management.md)) that includes relevant cross-domain decisions.

## Related Documents

- [Context management](context-management.md)
- [Agent task sizing](agent-task-sizing.md)
- [Git workflow](../delivery/version-control/git-workflow.md)
