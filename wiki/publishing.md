# Publishing as a GitHub Template

1. Create a GitHub repository and push the scaffold contents.
2. In repository settings, enable **Template repository** so users can create clean repositories without inheriting commit history.
3. Keep `main` internally consistent: agent entry files should remain short, local Markdown links should resolve, and skills should route into existing docs.
4. Add repository topics that describe agentic development, software architecture, product engineering, UX, and coding agents.
5. Use GitHub releases when the reusable playbook or workflow changes materially.
6. Keep product-specific examples under `examples/` so the scaffold itself remains reusable.

Users can then choose **Use this template**, name their product repository, put their idea in `.docs/project/brief.md`, and start a coding agent with `AGENTS.md` and `.docs/START.md`.
