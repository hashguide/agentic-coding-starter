# Customizing the Scaffold

## Add owner context
Use `.docs/project/constraints.md` for preferences, limits, required platforms, prohibited dependencies, business rules or existing systems.

## Add a playbook domain
Create focused documents under `.docs/playbook/<domain>/` with frontmatter fields `kind`, `domain`, `topics`, and `authority: guidance`. Add the route to `.docs/ROUTING.md` when the topic recurs across projects.

## Add stack-specific guidance
Keep vendor/framework implementation notes in focused playbook or guide documents. Project specs should record which technology was selected and why.

## Add agent adapters
Keep root agent files thin. An adapter should point the agent to `AGENTS.md` and `.docs/START.md`; shared operating logic belongs in `.docs/agent-system/`.

## Team-specific gates
Extend `MANIFEST.md`, templates, or CI checks to reflect regulatory, review or release requirements that apply across your projects.
