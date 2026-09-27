---
kind: playbook
domain: backend
topics: [backend, runtime, error, messages]
authority: guidance
---

# Error Messages

The server never tells the client more than the client needs to know. Error messages are security-relevant: a message that reveals which field failed, which record exists, or what the exact rate limit is gives an attacker information they can use. The server returns machine-readable error codes; the client decides what to display.

## Error Code Taxonomy

The server returns a structured object with a machine-readable `code` field. The client maps codes to user-facing strings — the server never produces user-facing copy directly.

```typescript
// Server response shape
type ErrorResponse = {
  code: string;      // Machine-readable — client maps this to display text
  errorId: string;   // Reference ID for support lookups
  fields?: Record<string, string[]>; // For VALIDATION_ERROR only — field names + violation types
};
```

**Standard error codes:**

| Code | HTTP status | Use for |
|---|---|---|
| `AUTH_REQUIRED` | 401 | No session or token present |
| `AUTH_FAILED` | 401 | Login failed (see auth enumeration rule) |
| `FORBIDDEN` | 403 | User is authenticated but lacks permission |
| `NOT_FOUND` | 404 | Resource not found (or not accessible — see below) |
| `VALIDATION_ERROR` | 400 | Input failed schema validation |
| `RATE_LIMITED` | 429 | Request rate exceeded |
| `CONFLICT` | 409 | Request conflicts with existing state (e.g., duplicate email on signup) |
| `INTERNAL_ERROR` | 500 | Unexpected server error |

Domain-specific codes extend this list: `INVOICE_NOT_FOUND`, `SUBSCRIPTION_REQUIRED`, `PLAN_LIMIT_REACHED`, etc.

## The Auth Enumeration Rule

**The server never distinguishes between a non-existent email and a wrong password.**

Both return HTTP 401 with code `AUTH_FAILED` and an identical response body. Nothing in the response reveals which field was wrong.

```typescript
async function login(email: string, password: string): Promise<Session> {
  const user = await db.user.findUnique({ where: { email } });

  // Do not short-circuit here with a different error for missing user
  // A timing attack could distinguish "user lookup fast" from "password check slow"
  const passwordValid = user
    ? await bcrypt.compare(password, user.passwordHash)
    : await bcrypt.compare(password, DUMMY_HASH); // Same timing regardless

  if (!user || !passwordValid) {
    throw new AuthError(); // Always AUTH_FAILED, always identical response
  }

  return createSession(user.id);
}
```

The timing attack mitigation (`bcrypt.compare` against a dummy hash when the user doesn't exist) ensures that the response time is the same whether the email exists or not, preventing timing-based enumeration even if the HTTP response is identical.

## The Account Existence Rule

**Signup endpoints never confirm whether an email is already registered.**

If a user attempts to sign up with an already-registered email, return `CONFLICT` — do not say "that email is already taken" in the response body. The safe response:

```typescript
async function signup(email: string, password: string): Promise<void> {
  try {
    await db.user.create({ data: { email, passwordHash: await bcrypt.hash(password, 12) } });
  } catch (err) {
    if (isDuplicateKeyError(err)) {
      // Do not throw here. Instead, send a "check your email" notification
      // to the email address. If it's already registered, they receive a
      // "someone tried to sign up with your email" notice. If it's a new
      // user at a shared email, they receive the verification link.
      // The HTTP response is the same either way: 200 with a generic message.
      await sendVerificationOrAlertEmail(email);
      return;
    }
    throw err;
  }
}
```

## Rate Limit Rule

Rate limit responses never reveal the limit parameters, window size, or remaining attempts.

```typescript
// WRONG: reveals implementation details
res.status(429).json({
  code: 'RATE_LIMITED',
  limit: 100,
  remaining: 0,
  resetAt: new Date(Date.now() + 60_000).toISOString(),
});

// CORRECT: machine-readable code only
res.status(429).json({
  code: 'RATE_LIMITED',
  errorId: crypto.randomUUID(),
});
```

## Error Code to Display Message Mapping

The client maintains this mapping. The server never produces the display string.

| Code | Safe user-facing message (client produces this) |
|---|---|
| `AUTH_FAILED` | "Incorrect email or password." |
| `AUTH_REQUIRED` | "Please sign in to continue." |
| `FORBIDDEN` | "You don't have permission to do this." |
| `NOT_FOUND` | "This item could not be found." |
| `VALIDATION_ERROR` | Per-field messages derived from the `fields` array |
| `RATE_LIMITED` | "Too many attempts. Please try again later." |
| `CONFLICT` | "This action could not be completed. Please try again." |
| `INTERNAL_ERROR` | "Something went wrong. Please try again." |

## Security

The server never reveals whether a value exists in the database. `AUTH_FAILED` means authentication failed — it does not indicate whether the email was found, whether the account is locked, or what the correct value might be. This prevents account enumeration attacks where an attacker determines which email addresses are registered by observing different error responses.

## Related Documents

- [Error handling](error-handling.md)
- [Auth session management](../../access/authentication/session-management.md)
- [Error feedback design](../../experience/interaction/error-feedback.md)
- [Auth pages layout](../../experience/layouts/auth-pages.md)
- [Error scenario testing](../../quality/testing/error-scenarios.md)
