---
kind: playbook
domain: access
topics: [access, authentication, mfa]
authority: guidance
---

# Multi-Factor Authentication

Multi-factor authentication (MFA) reduces account takeover risk by requiring proof from more than one factor. It is especially important for administrators, operators, financial actions, access to sensitive data, and accounts whose compromise affects other users.

## Factor Options

| Factor | Strength | Tradeoffs | Best fit |
|---|---|---|---|
| Passkey / WebAuthn | Phishing-resistant and device-bound | Recovery and cross-device support need careful UX | Preferred factor for users and administrators |
| Authenticator app (TOTP) | Offline, widely supported | Codes can be phished; clock drift and setup errors occur | Broad compatibility fallback |
| Hardware security key | Strong phishing resistance | Users must own and carry a key | Privileged and high-risk accounts |
| Push approval | Convenient when implemented by an identity provider | Push fatigue attacks; custom implementation is complex | Managed enterprise identity |
| SMS code | Better than password-only, but vulnerable to SIM swapping | Delivery failures, cost, and weaker assurance | Recovery or last-resort fallback only |
| Email code | Easy to deploy | Not a separate factor if email controls password recovery | Low-risk verification, not strong MFA |

## Enrollment and Challenge Flow

1. Require recent primary authentication before enrollment.
2. Register a passkey or show the TOTP secret as a QR code.
3. Require a successful challenge before marking the factor active.
4. Generate single-use recovery codes and ask the user to store them safely.
5. Notify the account through an independent channel that MFA changed.
6. Require MFA at sign-in and step up again before sensitive operations.

Do not treat enrollment as complete merely because a secret was generated. Persist activation only after verification succeeds.

## Recovery

Recovery is part of the security boundary, not a support shortcut. Provide several one-time recovery codes, store only salted hashes of them, consume each code atomically, and rotate the complete set after use or regeneration. A support agent must not disable MFA based only on information visible in an account profile.

For higher-risk products, recovery can require a second verified factor, an organization administrator, or a delayed process with user notification. Record enrollment, recovery, reset, and factor-removal events in the audit log without storing secrets or codes.

## Step-Up Authentication

A valid session does not prove that the person at the keyboard recently authenticated. Require a fresh MFA challenge for actions such as:

- Changing password, email, factors, or recovery settings
- Viewing or rotating API keys and secrets
- Changing roles or impersonating a user
- Initiating payouts, refunds, exports, or account deletion
- Entering an administrative area

Store the time and assurance level of the latest successful authentication in the server-side session. Do not trust a client-provided `mfaVerified` flag.

## Recommended Default

Prefer passkeys, support TOTP as a compatibility fallback, and issue recovery codes. Make MFA mandatory for administrators and other privileged roles. Offer it to all users, then require it where threat modeling or compliance demands it. SMS should not be the primary factor when phishing-resistant options are available.

## Security

- Rate-limit challenges by account and network without revealing whether an account exists.
- Encrypt factor credentials at rest; never log TOTP secrets, assertions, or recovery codes.
- Prevent replay and accept only a narrow TOTP time window.
- Bind challenges to the intended session, origin, user, and action.
- Invalidate other sessions after factor reset when the risk warrants it.
- Use generic failure responses and audit repeated failures.

## Related Documents

- [Auth options](options.md)
- [Session management](session-management.md)
- [Admin security](../../operations/admin/overview.md)
- [Audit log](../../operations/admin/audit-log.md)
- [Security checklist](../../security/checklist.md)
