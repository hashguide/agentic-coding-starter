---
kind: playbook
domain: agent-development
topics: [agent, development, prompt, library]
authority: guidance
---

# Prompt Library

Reusable prompt templates for common development tasks. Each template includes a reference to the relevant playbook documents the agent should read first, ensuring the output follows established conventions.

## Starting a New Feature

```
I'm implementing [feature name] in a TypeScript/Node.js backend with [ORM] and [framework].

Before writing any code, read:
- [relevant playbook document, e.g., "workers/webhooks.md" if this involves webhooks]
- [relevant playbook document, e.g., "backend/error-messages.md" for any API work]

Context:
- [Describe where this feature fits in the existing codebase]
- [Reference one or two existing files as style examples if useful]

Constraints:
- No raw SQL — ORM query builder only
- All error responses return structured error codes, not raw messages
- PII must not appear in logs
- [Add any feature-specific constraints]

Task:
[Describe exactly what to implement, with a bulleted acceptance criteria list]

Start with the schema/data layer. Show me that before moving to the API layer.
```

## Fixing a Bug

```
I have a bug in [file path, line number if known].

Observed behavior: [What's happening]
Expected behavior: [What should happen]
Reproduction steps: [Minimal steps to trigger it]

Relevant files:
- [file path]
- [file path]

Before fixing, explain what you believe is the root cause. Then propose the fix.
Do not change any code outside the scope of the bug fix.
Include a test that would have caught this bug if it existed before.
```

## Performing a Refactor

```
I want to refactor [specific area] for [reason: readability / performance / to match a pattern from the playbook].

Before refactoring, read:
- [relevant playbook document, e.g., "code-quality/module-boundaries.md"]

Current state: [brief description]
Target state: [brief description of the desired structure]

Rules:
- No behavior changes — this is a pure refactor
- All existing tests must continue to pass
- Do not expand scope to "clean up" adjacent code

Start by listing the files you'll change and what will change in each. Wait for my confirmation before writing any code.
```

## Reviewing a PR

```
Review the following diff for correctness. Pay specific attention to:

1. Does it follow the error response pattern in backend/error-messages.md?
   (Structured error codes, no raw error messages to clients, auth errors return AUTH_FAILED)
2. Are all database queries going through the ORM? No raw SQL strings?
3. Is there missing error handling for async operations?
4. Is the scope contained — no unrelated changes?
5. Are tests present and do they cover failure cases?

[Paste the diff]

Report findings as a numbered list. For each finding, note the file and line, what the issue is, and what the fix should be.
```

## Writing Tests

```
Write tests for [function/endpoint/component name] in [file path].

The test should cover:
- Happy path: [describe]
- [Failure case 1]: [describe]
- [Failure case 2]: [describe]
- [Edge case if relevant]: [describe]

Testing setup in this project:
- Test runner: [vitest / jest / pytest]
- Database: [use real test DB with transactional rollback / mock with X]
- External APIs: [mock with msw / use provider's test mode]

Follow the pattern in [path to an existing test file].
```

## Related Documents

- [Prompting for code](prompting-for-code.md)
- [Reviewing AI code](reviewing-ai-code.md)
- [Agent task sizing](agent-task-sizing.md)
- [Context management](context-management.md)
