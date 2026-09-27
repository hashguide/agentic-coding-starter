---
kind: playbook
domain: agent-development
topics: [agent, development, reviewing, ai, code]
authority: guidance
---

# Reviewing AI-Generated Code

The agent's summary of what it did is not a substitute for reading what it actually produced. Summaries are optimistic; diffs are ground truth. Every piece of agent-generated code must be reviewed before merging — the review checklist for AI-generated code is the same as for human-written code, plus a few AI-specific failure modes.

## Review Checklist

**Does it follow the relevant playbook document?**
If you referenced `backend/error-messages.md` in your prompt, verify the generated code actually returns structured error codes, not raw error strings. If you referenced `workers/webhooks.md`, verify the four-step handler pattern is implemented. Trust but verify.

**Does it introduce out-of-scope changes?**
AI agents frequently "clean up" or "improve" code that was not part of the task. Renamed variables, extracted functions, changed formatting — these are scope creep. They may be correct improvements, but they belong in a separate PR. Scope creep in AI-generated code is particularly insidious because it's easy to miss in a large diff.

**Does it follow error-message and logging rules?**
Check that: error responses return error codes, not raw messages; logs do not contain PII, tokens, or passwords; stack traces are not forwarded to API responses.

**Are tests present and meaningful?**
An agent that writes code without tests when asked to include them is giving you a summary ("here's the implementation") without the verification. An agent that writes tests that mock everything and only test the happy path is generating coverage theater. Check that tests exercise real failure cases.

**Does it match existing naming conventions?**
Agents frequently use their own naming preferences rather than matching the existing codebase. Check: file names (kebab-case?), function names (camelCase?), database column names (snake_case?), component names (PascalCase?).

## AI-Specific Red Flags

**Silently expanded scope:** The agent decided the task required changes to files you didn't mention, added a new dependency, or refactored a function "while I was in there." Each of these may be correct but must be conscious decisions, not surprises.

**Invented dependencies:** The agent imported a package that doesn't exist, used a method that doesn't exist on a real package, or referenced an environment variable that was never configured. See [handling-hallucination.md](handling-hallucination.md).

**Missing error handling:** The agent produced a "happy path" implementation that would crash on the first unexpected input, network error, or null value. Check every function for unhandled promise rejections, unchecked optionals, and missing try/catch where external calls are made.

**Plausible but wrong:** The code compiles and the tests pass, but the logic is subtly incorrect. This is the hardest failure mode to catch. It requires reading the code for semantic correctness, not just syntactic validity. Focus extra attention on: business rule implementations, access control logic, state machine transitions, and any calculation involving money.

## Process

1. Read the full diff before reading the agent's summary
2. Run the tests locally — don't trust CI alone for the first run of new test code
3. Spot-check each item in the checklist above
4. Verify any new dependency actually exists and is the package you think it is (`npm info <package-name>`)
5. For any security-adjacent code (auth, payments, input validation), review line by line

## Related Documents

- [Prompting for code](prompting-for-code.md)
- [Verification strategies](verification-strategies.md)
- [Handling hallucination](handling-hallucination.md)
- [Security checklist](../security/checklist.md)
