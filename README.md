# Agentic Product Scaffold

A reusable repository scaffold for developing a vague software idea into a specified, implemented, reviewed, and launchable product with coding agents.

The human owner supplies intent, constraints, preferences, domain knowledge, and approvals. Coding agents operate as the senior product engineers and technical leads: they identify missing requirements, compare appropriate approaches, design the product and system, write the specifications, implement them, critique the result, and keep project truth synchronized.

Agents are expected to research material unknowns when tools are available, use the manifest to identify applicable disciplines, and critique meaningful decisions until the project has an intentional answer across product requirements, IA/UX/UI, brand direction, architecture, data, security, accessibility, performance, observability, testing, delivery, and operations. Explicit owner decisions become constraints; routine professional decisions remain the agent's responsibility.

## Quick start

Use this repository as a GitHub template or clone it into a new project. Put whatever you already know in `.docs/project/brief.md`, then start your coding agent with:

```text
Read AGENTS.md and .docs/START.md.

Develop the idea in .docs/project/brief.md into a coherent product specification before implementation. Use the repository skills when available and route only the playbook guidance needed for each decision. Surface consequential choices for approval and write accepted conclusions into project and feature docs.
```

If your tool supports repository skills, it can progressively load the relevant workflow rather than receiving one large prompt.

## Knowledge model

```text
.agents/skills or tool adapter   repeatable workflow
             ↓
.docs/ROUTING.md                choose relevant context
             ↓
.docs/playbook/                 reusable options and decision guidance
             ↓
.docs/project/                  authoritative project-wide requirements
.docs/features/                 authoritative capability requirements
.docs/decisions/                durable consequential decisions
             ↓
implementation
             ↓
.docs/validation/               review findings
```

Canonical skills are stored in `.agents/skills/`; Claude Code adapters are in `.claude/skills/`.

## Documentation structure

```text
.docs/
├── START.md                  # agent entrypoint
├── ROUTING.md                # task → relevant context
├── MANIFEST.md               # specification coverage model
├── agent-system/             # context, documentation, decisions and collaboration rules
├── playbook/                 # reusable options, tradeoffs and implementation guidance
├── templates/                # feature, decision and review templates
├── project/                  # authoritative project-wide specification
├── features/                 # authoritative feature specifications
├── decisions/                # durable architectural/product decisions
├── execution/                # roadmap, current work and risks
└── validation/               # saved review findings
```

The operating rule is:

**Route to guidance → make a project decision → record project truth → implement → validate → update project truth.**

Agents should not read the entire playbook. Skills and `ROUTING.md` provide progressive context discovery, while project and feature docs preserve decisions so later agents do not need to rediscover them.

## Documentation authority

For implementation decisions, use this order:

1. Current explicit task constraints
2. Accepted project specifications in `.docs/project/`
3. Relevant feature specifications in `.docs/features/`
4. Accepted decision records in `.docs/decisions/`
5. Reusable guidance in `.docs/playbook/`
6. Existing implementation

Existing code describes the current system; project truth defines intended behavior.

## Learn the workflow

- [`wiki/getting-started.md`](wiki/getting-started.md) — start a new project
- [`wiki/how-it-works.md`](wiki/how-it-works.md) — knowledge layers and routing
- [`wiki/skills.md`](wiki/skills.md) — workflow skills and progressive disclosure
- [`wiki/workflows.md`](wiki/workflows.md) — discovery, specification, implementation and review loops
- [`wiki/prompts.md`](wiki/prompts.md) — small prompts for common entry points
- [`wiki/customizing.md`](wiki/customizing.md) — adapt the scaffold to a team or stack
- [`wiki/agent-support.md`](wiki/agent-support.md) — agent-file adapters and context use
- [`examples/idea-to-product.md`](examples/idea-to-product.md) — example progression from vague idea to project truth

## Design principle

The playbook is broad; project context stays narrow. Agents load only what applies, resolve relevant concerns as specified, deferred, or not applicable, and keep the resulting decisions in project truth rather than repeatedly feeding the full knowledge library into context.

## License

MIT. See [`LICENSE`](LICENSE).
