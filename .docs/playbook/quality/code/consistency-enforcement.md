---
kind: playbook
domain: quality
topics: [quality, code, consistency, enforcement]
authority: guidance
---

# Consistency Enforcement

Code style consistency cannot be maintained by convention alone. Conventions are forgotten under deadline pressure, applied inconsistently by different contributors (human and AI), and impossible to audit at scale. Automated tooling enforces consistency before code reaches the repository, turning style compliance from a manual review burden into a guaranteed property.

## ESLint and Prettier

**Prettier** handles formatting: indentation, line length, quote style, semicolons, trailing commas. It is opinionated and does not allow bikeshedding — you configure it once and it enforces universally.

**ESLint** handles code quality rules: unused variables, missing dependencies in useEffect, forbidden patterns.

Do not configure both to handle the same concerns. Use Prettier for all formatting; use ESLint for all logic-level rules. Use `eslint-config-prettier` to disable ESLint's formatting rules so they don't conflict.

```bash
npm install -D prettier eslint eslint-config-prettier
```

`.prettierrc`:
```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100
}
```

`.eslintrc.json`:
```json
{
  "extends": ["eslint:recommended", "plugin:@typescript-eslint/recommended", "prettier"],
  "rules": {
    "no-unused-vars": "off",
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "warn"
  }
}
```

## TypeScript Strict Mode

TypeScript's strict mode enables a family of checks that catch entire categories of bugs at compile time.

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

**What `strict: true` enables:** `strictNullChecks` (prevents accessing possibly-undefined values), `strictFunctionTypes`, `strictPropertyInitialization`, `noImplicitAny`. These eliminate the most common TypeScript-while-actually-JavaScript patterns.

**Enable this from day one.** Enabling strict mode on an existing codebase requires fixing every type error before the compiler accepts the change. Day-one strict mode prevents this debt.

## Pre-Commit Hooks with Husky and lint-staged

A pre-commit hook runs automatically before every commit and blocks the commit if checks fail. This is the enforcement mechanism — without it, lint and format rules are advisory.

```bash
npm install -D husky lint-staged
npx husky init
```

`.husky/pre-commit`:
```bash
npx lint-staged
```

`package.json`:
```json
{
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md,css}": [
      "prettier --write"
    ]
  }
}
```

lint-staged runs only on staged files, not the entire codebase — so the pre-commit hook runs in under 5 seconds even on large projects.

**The commit is blocked if ESLint reports an error.** Warnings pass; errors do not. Configure rules at the `error` level for violations that must never reach the repository, and `warn` level for things you want to notice but not enforce strictly.

## Related Documents

- [Naming conventions](naming-conventions.md)
- [Dead code and cleanup](dead-code-and-cleanup.md)
- [Git workflow](../../delivery/version-control/git-workflow.md)
