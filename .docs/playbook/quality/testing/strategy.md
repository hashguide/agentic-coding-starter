---
kind: playbook
domain: quality
topics: [quality, testing, strategy]
authority: guidance
---

# Testing Strategy

A testing strategy defines what to test, at what layer, and in what proportion. The goal is maximum confidence in the critical paths at minimum maintenance cost.

## The Testing Pyramid

```
        ┌─────────┐
        │   E2E   │  ← Few: 5–10 critical user flows
        │  Tests  │     Playwright or Cypress
        ├─────────┤
        │Integra- │  ← Some: API endpoints, DB queries
        │  tion   │     Real test DB, supertest
        │  Tests  │
        ├─────────┤
        │  Unit   │  ← Many: Business logic, validation, utilities
        │  Tests  │     Jest, pure functions, fast
        └─────────┘
```

**Unit tests** are fast, cheap to write, and test in isolation. Write many.
**Integration tests** test that real components work together. Write some.
**E2E tests** test the user-visible flows through the real UI. Write a few, focused on the highest-value paths.

## Proportion Guidance

For a typical SaaS product:
- **Unit:** 60–70% of tests — every business logic function, validation schema, utility
- **Integration:** 20–30% — every API endpoint, every background job, every DB query pattern
- **E2E:** 5–10% — signup → onboarding → core action, upgrade flow, account deletion

If you have 100 tests: ~65 unit, ~25 integration, ~10 E2E. If you have 10 tests, 2 E2E is enough.

## Coverage Percentage vs. Critical Path Coverage

**Coverage percentage is a weak signal.** 90% line coverage is meaningless if the 10% uncovered includes your payment processing and auth logic. A codebase with 60% coverage that covers signup, checkout, and the core workflow is more confident than one with 90% coverage of getters and utilities.

**What to prioritize:**
1. Auth flows (signup, login, password reset, MFA)
2. Payment flows (checkout, subscription creation, webhook handling)
3. Core product workflow (whatever your product's primary value action is)
4. Account deletion and data export (high-stakes, infrequently tested manually)

Coverage can help identify dead code paths and untested branches, but treat it as a diagnostic tool, not a goal.

## What Each Layer Tests

**Unit tests test:** Pure logic that doesn't touch external state. Business rules (`canUserAccess(plan, feature)`), validation schemas (Zod), data transformation utilities, error code mappings.

**Integration tests test:** Real components working together. An API endpoint that validates input, queries the DB, and returns the correct response. A background job that sends an email. A webhook handler that updates a subscription.

**E2E tests test:** User-visible flows in a real browser. Signup → email verification → onboarding → first action. Checkout flow. Error states visible to the user.

## Related Documents

- [Testing unit](unit.md)
- [Testing integration](integration.md)
- [Testing E2E](e2e.md)
- [Testing error scenarios](error-scenarios.md)
