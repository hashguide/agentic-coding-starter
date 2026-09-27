---
kind: playbook
domain: quality
topics: [quality, code, dead, and, cleanup]
authority: guidance
---

# Dead Code and Cleanup

Dead code is any code that is no longer executed in any reachable path — orphaned functions, stale feature flags, commented-out blocks, unused exports, and abandoned experimental branches. It accumulates silently and creates real costs: it confuses readers, misleads AI agents, inflates build times, and makes refactoring harder because every deletion requires verifying that something isn't secretly still in use.

## Identifying Genuinely Dead Code

**Static analysis — TypeScript and ESLint:**

```bash
# TypeScript's noUnusedLocals and noUnusedParameters catch unused variables and parameters
# Add to tsconfig.json:
{
  "compilerOptions": {
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}

# ESLint's no-unused-vars catches unused imports
# Already covered if you run ESLint with the recommended ruleset
```

**Finding unused exports:**

```bash
# ts-prune finds exported symbols that are never imported anywhere
npx ts-prune
```

**Code that looks unused but isn't:**
- Dynamic imports (the string might not be greppable)
- Exports consumed by generated code (ORM types, GraphQL schemas)
- Functions called via reflection or dependency injection containers
- `_`-prefixed parameters (intentionally unused in callbacks)

Before deleting something that static analysis flags, do a manual search to confirm:

```bash
grep -r "functionName" src/ --include="*.ts" --include="*.tsx"
```

## Stale Feature Flags

Feature flags that have been fully rolled out and are now `true` for 100% of users are dead code. The conditional branch that would be taken if the flag were `false` is unreachable.

**Cleanup process:**
1. Confirm the flag is 100% enabled in production and has been for at least one release cycle
2. Remove the flag check and keep the enabled code path
3. Remove the disabled code path entirely
4. Remove the flag from the admin feature flags configuration
5. Remove the flag from the flag registry/schema

Do not leave flag checks in place "just in case" — this defeats the purpose of feature flags and accumulates indefinitely.

## Commented-Out Code

Commented-out code should be deleted, not preserved. The git history is the recovery mechanism — if the code was useful, it will be findable by looking at git history. Commented-out code in the active codebase:
- Misleads readers about what code is active
- Misleads AI agents into thinking the commented approach is an alternative worth considering
- Is never actually restored — it just persists until someone works up the motivation to delete it

**Exception:** A comment that says `// Can't use X here because of Y` without showing the old code is a legitimate warning. That's a comment about a constraint, not commented-out code.

## Recommended Cleanup Cadence

Rather than a periodic "cleanup sprint," integrate cleanup into the normal development flow:

- **Per PR:** Delete dead code in files you're already touching (Boy Scout Rule from [refactoring-triggers.md](refactoring-triggers.md))
- **Monthly:** Run `ts-prune` and address the top 10 unused exports
- **Per feature flag:** Immediately clean up when a flag reaches 100% rollout

A dedicated cleanup sprint is usually a sign that the Boy Scout Rule hasn't been applied. Use cleanup sprints to recover, then prevent recurrence with continuous small deletions.

## Related Documents

- [Refactoring triggers](refactoring-triggers.md)
- [Consistency enforcement](consistency-enforcement.md)
- [Admin feature flags](../../operations/admin/feature-flags.md)
