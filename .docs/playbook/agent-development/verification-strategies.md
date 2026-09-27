---
kind: playbook
domain: agent-development
topics: [agent, development, verification, strategies]
authority: guidance
---

# Verification Strategies

An agent's summary of what it did is a claim, not a proof. "The tests pass" means the agent believes the tests pass. "The migration was applied cleanly" means the agent ran a command and interpreted the output as success. Independent verification is not distrust — it's the difference between knowing and assuming.

## Run the Test Suite Yourself

After an agent implements a feature with tests, run the tests locally — do not rely on the agent's report of the test results. Agents occasionally:
- Write tests that pass by mocking the thing being tested
- Omit a test for a case they consider obvious
- Report tests as passing when they actually skipped

```bash
npm test
# or
npx vitest run
# or
pytest
```

Check the output: how many tests ran? Did any skip? Did the coverage change in a way that makes sense? A "passing" suite that ran 3 fewer tests than before needs an explanation.

## Review the Full Diff

Read the diff before reading the agent's summary. The agent's summary describes the intended change; the diff describes the actual change.

```bash
git diff main...HEAD
# or review in your editor's source control view
```

Look for:
- Files changed that you didn't ask the agent to touch
- New dependencies added to package.json
- Environment variables referenced but not documented
- Import statements for packages you haven't seen in this project

## Claim Types That Always Warrant Independent Verification

**Tests passing:** Run them yourself. This is non-negotiable.

**Migrations applied cleanly:** Apply the migration in a fresh local database, not just on the already-migrated development database. Run `npx prisma migrate reset && npx prisma migrate dev` (or equivalent) to verify from a clean state.

**External API calls succeeding:** If the agent says "I tested the Stripe integration," confirm by checking the Stripe dashboard for a test event, or by reviewing the actual network log, not the agent's description of it.

**Security controls working:** If the agent says "the endpoint requires authentication," verify by sending a request without a valid session cookie and confirming you get a 401. Don't accept "I added the auth middleware" as verification.

**Webhook signature verification working:** Send a request with a wrong signature to the webhook endpoint. Verify it returns 400, not 200.

**Edge cases handled:** If the agent says "error handling is in place," trigger the error. Pass null where a value is expected. Send a malformed payload. Actually see the error handler fire.

## When to Trust Without Full Verification

Styling changes and documentation changes with no logic are lower risk and require less verification than business logic, security controls, and external integrations. Apply proportional scrutiny: more scrutiny for code that handles money, authentication, or PII; less for CSS tweaks and README updates.

## Related Documents

- [Reviewing AI code](reviewing-ai-code.md)
- [Handling hallucination](handling-hallucination.md)
- [Testing strategy](../quality/testing/strategy.md)
- [Error scenarios testing](../quality/testing/error-scenarios.md)
