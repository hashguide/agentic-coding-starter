# Agent Support

`AGENTS.md` is the canonical short entrypoint. Tool-specific files such as `CLAUDE.md`, `GEMINI.md`, Cursor rules, and GitHub Copilot instructions delegate to it instead of duplicating the operating model.

Workflow skills provide progressive disclosure for tools that support them:

- canonical skills: `.agents/skills/`
- Claude Code adapters: `.claude/skills/`

When adding another coding agent:
1. create the smallest supported instruction file for that tool;
2. point it to `AGENTS.md` and `.docs/START.md`;
3. expose the canonical workflow skills through the tool's native skill mechanism when practical;
4. keep product-specific context in `.docs/project/` and `.docs/features/`;
5. keep reusable decision knowledge in `.docs/playbook/`.

Avoid copying the playbook into tool-specific instruction files. Fresh sessions should receive a map and discover deeper context only when the task needs it.
