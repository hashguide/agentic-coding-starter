---
kind: playbook
domain: agent-development
topics: [agent, development, context, management]
authority: guidance
---

# Context Management

An AI coding session has a context window — a limit to how much text the model can hold in active consideration. As a session grows longer, earlier context gets pushed out or compressed, the model loses access to earlier decisions, and responses begin to drift. Managing context deliberately is the difference between a session that produces coherent, consistent code and one that produces contradictions.

## Context Budget Awareness

Most models have context windows between 100,000 and 200,000 tokens (roughly 75,000–150,000 words). A session burns context from multiple sources:
- Your messages
- The model's responses
- Pasted code, logs, and file contents
- Tool call results

**Rule of thumb:** A session becomes unreliable after 60–70% of the context window is consumed. You will notice this as: the model forgetting a constraint you mentioned early in the session, repeating code you already wrote, or producing solutions that contradict earlier decisions.

## Signs of Accumulated Stale Context

- The model starts suggesting a library you already decided against
- It repeats an explanation it gave you 20 messages ago
- It writes code that contradicts the error format you established earlier
- Responses feel generic and less specifically tuned to your codebase
- It "forgets" that you're using TypeScript and writes JavaScript

When you notice any of these signs, start a fresh session rather than trying to correct within the current one.

## Summarizing Decisions Instead of Re-Pasting Files

Before starting a fresh session, produce a decision summary — a concise paragraph of the choices already made. This is more effective than re-pasting full files because it gives the model the *conclusions* without consuming context on the *evidence*.

**Example decision summary:**
> "For this project: TypeScript/Node.js backend with tRPC, Prisma ORM on PostgreSQL. Auth is via Clerk. Error responses follow the error code taxonomy in `backend/error-messages.md`. No raw SQL. Background jobs use BullMQ with Redis. We decided against GraphQL. The notification system uses server-side persistence (notifications table in Postgres), not localStorage. Feature flags are simple database-backed boolean flags per user, not a third-party service."

Paste this at the start of every new session. It re-establishes the constraints in under 500 tokens.

## One Session Per Feature/Domain

Start fresh sessions per logical unit of work — per feature, per domain boundary, per significant architectural decision. Do not maintain one long-running session across a full day of development across multiple features.

**Rationale:** A session that starts with "build the auth system" and ends six hours later having also touched payments, admin, and email has accumulated context from all four domains. Every domain in the session degrades the quality of every other.

**Session boundaries:**
- Starting a new feature → new session
- Moving from backend to frontend work → new session
- After a large refactor → new session
- Any time the model seems to be forgetting earlier decisions → new session

## What to Include in Context

Include at the start of each session:
- Decision summary (see above)
- The specific task for this session
- The relevant playbook documents by path (if using a file-aware agent)
- At most one or two representative existing files for style/pattern reference

Do not include:
- Full contents of files the task doesn't touch
- Long conversation history from other sessions
- Logs and stack traces unless debugging a specific error

## Related Documents

- [Prompting for code](prompting-for-code.md)
- [Agent task sizing](agent-task-sizing.md)
- [Multi-agent workflows](multi-agent-workflows.md)
