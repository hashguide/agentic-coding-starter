---
kind: playbook
domain: experience
topics: [experience, layouts, auth, pages]
authority: guidance
---

# Auth Pages

Auth pages — login, signup, and password reset — are the first experience authenticated users have with the product. They must be fast, minimal, and secure. Friction here means lost users; security failures here mean compromised accounts.

## Signup Form

Minimize the fields required to create an account. Ask only for what is needed to create the account and send the first communication:

```
[Product logo]

Create your account

[Email input]
[Password input]
──────────────
[Create account] ← primary CTA, specific label

──── or ────

[Continue with Google]
[Continue with GitHub]

Already have an account? [Sign in]
```

**What to ask at signup:**
- Email address (required)
- Password (required if not using social login only)

**What not to ask at signup:**
- Full name — defer to onboarding
- Company name — defer to onboarding
- Phone number — defer to onboarding, and only if genuinely needed
- Payment information — never at signup unless the product is paid-only with no trial

**Why:** Each additional field at signup reduces conversion. Collect the minimum to create the account. Collect the rest during onboarding when the user has already committed.

## Login Form

```
[Product logo]

Welcome back

[Email input]
[Password input]  [Forgot password?] ← link, right-aligned

[Sign in] ← primary CTA

──── or ────

[Continue with Google]

Don't have an account? [Sign up]
```

## Login Error Message Rule

A failed login always shows: **"Incorrect email or password."**

This message is shown whether the email address doesn't exist in the database or the password is wrong. The UI must never distinguish between these two cases.

**What not to show:**
- "That email isn't registered" — reveals which emails exist
- "Wrong password" — reveals the email exists
- "Account not found" — same problem

The server enforces this by always returning `AUTH_FAILED` for both cases. The client maps `AUTH_FAILED` to the single display string "Incorrect email or password." regardless of any other context.

## Rate Limit UI Feedback

When login is rate-limited, display:
**"Too many attempts. Please try again later."**

**Never reveal:**
- When the limit resets ("Try again in 15 minutes")
- What the limit is ("You've made 10 attempts")
- What the parameters are

Revealing the rate limit window tells an attacker exactly when to resume. The generic message is intentionally vague.

## Password Reset Flow

**Step 1 — Request:**
```
[Email input]
[Send reset link]
```

Response (always the same, whether email exists or not):
> "If that email is registered, you'll receive a reset link shortly."

This prevents email enumeration via the reset form.

**Step 2 — Reset link sent email:**
Contains a time-limited link (24-hour expiry). The link includes a signed token stored in the database.

**Step 3 — New password form:**
```
[New password]
[Confirm new password]
[Set new password]
```

Invalidate the token after use. Invalidate all other active sessions on successful reset (security best practice — existing sessions may have been the result of the compromised password).

## Security

**The login UI must display identical error messages whether the email doesn't exist or the password is wrong.** Distinguishing them tells an attacker which emails are registered. This is an enumeration vulnerability.

**The password reset UI must display the same response whether the email is registered or not.** The same reason: an attacker should not be able to enumerate registered emails by submitting them to the reset form.

## Related Documents

- [Backend error messages (AUTH_FAILED rule)](../../backend/runtime/error-messages.md)
- [Error feedback (client mapping table)](../interaction/error-feedback.md)
- [Auth session management](../../access/authentication/session-management.md)
- [Testing error scenarios](../../quality/testing/error-scenarios.md)
