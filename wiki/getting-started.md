# Getting Started

## 1. Create a project

Use this repository as a GitHub template or clone it into a new repository.

## 2. Add what you know

Write the initial idea in `.docs/project/brief.md`. Add hard preferences, constraints, required technologies, or explicit product choices to `.docs/project/constraints.md`.

The idea can be incomplete. The scaffold is designed for the coding agent to develop it into a product specification.

## 3. Start the coding agent

A minimal starting instruction is:

```text
Read AGENTS.md and .docs/START.md. Develop the idea in .docs/project/brief.md into a complete first-release product specification. Use repository skills when available, route only the playbook guidance needed for each decision, research material unknowns, and surface consequential product choices for my approval.
```

## 4. Let the agent own professional decisions

The agent should identify missing requirements and resolve routine decisions across product strategy, competitive expectations, journeys, information architecture, UX/UI, brand direction, frontend/backend architecture, data, security, accessibility, performance, analytics, observability, testing, deployment, and operations.

Use `.docs/MANIFEST.md` as the completeness gate. Each applicable area should become specified, intentionally deferred, or not applicable with rationale.

## 5. Approve consequential choices

Expect the agent to bring you decisions that materially affect product direction, scope, cost, risk, or difficult-to-reverse commitments. You can also state any decision directly; record it in project constraints or the relevant specification.

## 6. Build and review

Once a feature is specified, use the implementation workflow. Meaningful work should then receive a product review that inspects the actual behavior and rendered interface where available.
