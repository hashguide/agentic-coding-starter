---
kind: playbook
domain: agent-development
topics: [agent, development]
authority: guidance
---

# AI-Assisted Development Overview

This directory covers how to work effectively with an AI coding agent during development. It is distinct from `AGENT.md`, which governs how an agent navigates and generates content within this playbook itself. This directory governs the human-agent collaboration process when building the actual product.

## The Distinction from AGENT.md

`AGENT.md` is a navigation document: it tells an AI agent how to find the right playbook document when a user asks a product-building question. It is a reference index for this library.

This directory is operational: it covers prompting technique, context management, code review discipline, task sizing, and how to verify that what the agent produced is actually correct. It applies every time you open a coding session with an AI assistant.

## What This Directory Covers

- [prompting-for-code.md](prompting-for-code.md) — Structuring prompts that produce usable code
- [context-management.md](context-management.md) — Managing the context window across a session
- [reviewing-ai-code.md](reviewing-ai-code.md) — What to check before merging agent-generated code
- [agent-task-sizing.md](agent-task-sizing.md) — Right-sizing tasks given to an agent
- [verification-strategies.md](verification-strategies.md) — Independently verifying agent claims
- [handling-hallucination.md](handling-hallucination.md) — Catching invented APIs and fake packages
- [multi-agent-workflows.md](multi-agent-workflows.md) — Coordinating multiple agent sessions
- [prompt-library.md](prompt-library.md) — Reusable templates for common tasks
- [ai-pair-programming.md](ai-pair-programming.md) — Working style options alongside an agent

## Related Documents

- [Code quality overview](../quality/code/overview.md)
- [Reviewing AI code](reviewing-ai-code.md)
- [Verification strategies](verification-strategies.md)
