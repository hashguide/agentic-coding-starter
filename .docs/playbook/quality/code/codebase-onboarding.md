---
kind: playbook
domain: quality
topics: [quality, code, codebase, onboarding]
authority: guidance
---

# Codebase Onboarding

Whether the new contributor is a human developer or an AI agent starting a fresh session, the first task is to build an accurate mental model of the codebase before making any changes. A new contributor who skips this step will invent conventions, contradict established patterns, and create code that looks foreign to the rest of the codebase.

## Recommended Reading Order

Follow this sequence before opening any file for editing:

1. **README.md** — The project purpose, how to run it locally, and how to run tests. This is the orientation document.

2. **AGENT.md or CLAUDE.md** — The conventions document: project-specific patterns, anti-patterns, tooling choices, and instructions that override defaults. For AI agents, this file is the most important onboarding document.

3. **Folder structure** — Read the top-level directory structure. Don't open files yet — just understand the organization. Does it use feature-based structure? Domain-based? Layered? The structure reveals the architecture.
   ```bash
   find . -type d -not -path '*/node_modules/*' -not -path '*/.git/*' | head -30
   ```

4. **One representative file per layer** — Read one file from each layer: one route handler, one service, one database query file, one React component, one test file. This reveals the naming conventions, error handling patterns, logging approach, and code style actually in use — not just what the docs claim.

5. **Then, and only then, write code.**

## The Rule Against Inventing New Conventions

Before introducing a new pattern — a new utility function structure, a new way of organizing imports, a new error type — confirm that an existing pattern doesn't already cover the case.

```bash
# Search for existing patterns before creating new ones
grep -r "function createNotification" src/
grep -r "ValidationError" src/
grep -r "createRouter" src/
```

Inventing a redundant pattern creates two ways to do the same thing, which means future contributors (and AI agents) cannot infer the correct approach from reading the code.

**When to introduce a new pattern:** When you've confirmed via search that no existing pattern covers the case, and when the new pattern is consistent with the established conventions (same naming style, same error handling approach, same logging approach).

## What a Good Onboarding Reveals

After completing the reading sequence, you should be able to answer:

- What database ORM is used, and what does a typical query look like?
- How are errors represented and returned from API endpoints?
- What does the test setup look like (test database, fixtures, mocking strategy)?
- What logging library is used, and what fields does a typical log entry include?
- How is authentication checked in route handlers?
- What does a typical React component look like (hooks, data fetching, error states)?

If you cannot answer these questions after the reading sequence, read one more representative file per question before writing code.

## For AI Agents

When starting a new session:

1. Read the CLAUDE.md or AGENT.md file first
2. Run the find command to see existing structure
3. Read one example file from the area you'll be working in
4. State the conventions you've identified before writing any code — this surfaces misunderstandings before they become wrong code

## Related Documents

- [Naming conventions](naming-conventions.md)
- [Module boundaries](module-boundaries.md)
- [AI development overview](../../agent-development/overview.md)
- [Context management](../../agent-development/context-management.md)
