---
kind: playbook
domain: quality
topics: [quality, code]
authority: guidance
---

# Code Quality Overview

Codebases don't degrade suddenly — they degrade gradually, through a sequence of small compromises that each seem reasonable in isolation. A boundary gets blurred once because it was faster. A name gets chosen carelessly because nobody was looking. Dead code accumulates because removing it felt risky. Over time, these small decisions compound into a codebase that slows every subsequent decision.

## The Common Decay Pattern

**Blurred boundaries:** A module that was once responsible for one thing starts handling two, then three. Callers that should go through a public interface start reaching into internals directly. Eventually, you can't change anything without checking every caller.

**Inconsistent naming:** Different parts of the codebase use different conventions for the same concepts. `user_id`, `userId`, `UserId`, and `uid` appear in different files. Every new developer (or AI agent) must learn the local dialect of each file rather than reading fluently across the whole codebase.

**Accumulating dead code:** Features are removed but their helpers, types, and utilities remain. Feature flags are never cleaned up. Commented-out code persists because it "might be useful." The reader cannot tell what's active and what's vestigial.

## How This Directory Counters Each Cause

- [naming-conventions.md](naming-conventions.md) — eliminates the naming inconsistency cause
- [module-boundaries.md](module-boundaries.md) — prevents boundary blurring before it starts
- [dependency-direction.md](dependency-direction.md) — prevents circular and inverted dependencies
- [function-and-file-size.md](function-and-file-size.md) — catches "doing too many things" before it compounds
- [code-smells.md](code-smells.md) — naming the specific patterns that signal decay
- [refactoring-triggers.md](refactoring-triggers.md) — tells you when to fix it and when to leave it
- [dead-code-and-cleanup.md](dead-code-and-cleanup.md) — actively removes accumulated vestigial code
- [consistency-enforcement.md](consistency-enforcement.md) — automates the enforcement that humans forget
- [documentation-in-code.md](documentation-in-code.md) — keeps code self-documenting
- [codebase-onboarding.md](codebase-onboarding.md) — ensures every new reader starts with the right mental model

## Related Documents

- [Engineering principles](../../product/engineering-principles.md)
- [Architecture overview](../../architecture/overview.md)
