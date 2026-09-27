---
kind: playbook
domain: agent-development
topics: [agent, development, ai, pair, programming]
authority: guidance
---

# AI Pair Programming

There are three distinct working modes when developing alongside an AI agent, each suited to a different kind of task. Using the wrong mode for the task creates friction: putting the agent in the lead on judgment-heavy decisions produces code you don't understand; keeping the human in the lead on well-specified mechanical tasks is slower than necessary.

## Mode 1: Agent Leads

The agent writes, you review. You provide a well-specified prompt; the agent produces a complete implementation; you read, verify, and merge or request changes.

**Best for:**
- Well-specified tasks with clear acceptance criteria
- Boilerplate-heavy work (CRUD endpoints, migration files, test fixtures)
- Tasks in areas where the playbook has a clear prescribed pattern (webhook handlers, error response format)
- Any task where you can precisely describe what "done" looks like

**Your role:** Prompt precisely, verify independently (see [verification-strategies.md](verification-strategies.md)), and review the diff carefully before merging.

**Risk:** Agent makes plausible-but-wrong architectural decisions in areas where you didn't specify the constraint. Mitigated by good prompts and thorough review.

## Mode 2: Human Leads

You write, the agent assists, explains, and suggests. You maintain the keyboard; the agent answers questions, explains unfamiliar APIs, suggests completion patterns, and catches errors you point to.

**Best for:**
- Exploratory work where you're not sure what the implementation should look like
- Learning a new library, framework, or pattern — the agent explains as you go
- Judgment-heavy decisions (system design, data model shape, access control policy)
- Areas where subtle errors have high stakes (auth, payments, PII handling)

**Your role:** Make the structural decisions; use the agent as a well-informed collaborator who can explain tradeoffs but doesn't drive.

**Risk:** Slower than agent-leads for tasks that were actually well-specified. Don't use this mode for tedium.

## Mode 3: True Pairing (Alternating Turns)

You and the agent take alternating turns: you write a test, the agent makes it pass; the agent proposes a function signature, you critique and refine; you outline the algorithm, the agent fills in the implementation. Each party sees and builds on the other's output.

**Best for:**
- Test-driven development — write tests first, agent fills implementation
- Design-by-contract — you define the interface, agent implements it
- Refactoring — you specify the target structure, agent moves the code

**Your role:** Set the contract, write the tests, define the structure. The agent executes within those constraints.

**Risk:** Requires active engagement throughout — this is not "agent does everything while I watch." The value is in the division of labor: you own the design decisions, the agent owns the mechanical execution.

## Choosing a Mode

| Task type | Recommended mode |
|---|---|
| Well-specified CRUD endpoint | Agent leads |
| New domain area you don't fully understand | Human leads |
| TDD implementation cycle | True pairing |
| Architectural decision | Human leads |
| Writing boilerplate tests for existing code | Agent leads |
| Debugging a subtle logic error | Human leads |
| Refactoring with a clear target structure | True pairing |

## Related Documents

- [Prompting for code](prompting-for-code.md)
- [Agent task sizing](agent-task-sizing.md)
- [Reviewing AI code](reviewing-ai-code.md)
