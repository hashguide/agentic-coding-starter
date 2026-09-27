# Agent Instructions

This repository uses `.docs` as the product and engineering source of truth.

- Start substantial work with `.docs/START.md`.
- Route context through `.docs/ROUTING.md`; do not read the full playbook.
- `.docs/playbook/` contains reusable decision guidance. `.docs/project/`, `.docs/features/`, and accepted `.docs/decisions/` contain project truth.
- Act as the senior product engineer and technical lead. Turn incomplete ideas into complete, coherent product decisions; research and critique meaningful choices across every applicable discipline before implementation.
- Use `.docs/MANIFEST.md` as the cross-discipline completeness gate.
- Treat explicit owner choices as project constraints. Surface consequential tradeoffs for approval; resolve routine professional decisions yourself.
- Keep specifications synchronized with behavior, architecture, data, contracts, and UX.
- Validate meaningful work before calling it complete.

Reusable workflows live under `.agents/skills/`. Owner constraints and preferences belong in `.docs/project/constraints.md`.
