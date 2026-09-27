---
kind: playbook
domain: access
topics: [access, authentication, options]
authority: guidance
---

# Auth Options

Authentication is security-critical code that developers routinely get wrong: insecure password storage, missing CSRF protection, JWT misconfiguration, incomplete logout. Managed auth providers handle these correctly by default. Building custom auth means taking on responsibility for every one of these failure modes.

## Managed Providers vs. Custom Auth

**The recommendation:** Most solo builders should use a managed auth provider rather than custom auth. The exceptions are products with unusual auth requirements, strict data residency constraints that prohibit third-party user management, or teams with specific security expertise.

The cost of a managed provider (pricing) is almost always lower than the cost of building and maintaining secure custom auth (engineering time + incident response).

## Provider Comparison

| Provider | Cost | Control | Setup time | Key strengths | Limitations |
|---|---|---|---|---|---|
| **Clerk** | Free up to 10K MAU; $25/mo after | Medium | Very low (< 1 hour) | Best-in-class prebuilt UI components; organizations/teams built-in | Higher cost at scale; US-hosted by default |
| **Auth0** | Free up to 7,500 MAU; scales up | High | Low–Medium | Enterprise features, highly configurable, many integrations | More complex to configure; can be expensive |
| **Supabase Auth** | Free tier; pay-per-project beyond | High | Low | Integrated with Supabase DB; open-source; self-hostable | Fewer prebuilt UI components |
| **Lucia** | Free (library) | Full | Medium–High | Full control; no vendor dependency; TypeScript-native | You own the security implementation |
| **NextAuth.js / Auth.js** | Free (library) | Full | Medium | Well-tested patterns; many providers built-in | Less opinionated; more configuration required |

## When Each Makes Sense

**Clerk:** You want to move fast and have prebuilt auth UI (sign-in, sign-up, user profile components). You're building a team/organization-aware product. You don't have strict data residency requirements.

**Auth0:** You need advanced customization (custom rules, enterprise SSO, compliance certifications), or you're building a product that will eventually need SAML/enterprise identity.

**Supabase Auth:** You're already using Supabase as your database. The integration is seamless and the row-level security model pairs naturally with Supabase's RLS.

**Lucia or NextAuth:** You need full control over the implementation, cannot use third-party user management for compliance reasons, or you have time to build and maintain the auth layer correctly.

## What "Custom Auth" Actually Requires

If you choose a library (Lucia, Auth.js) rather than a managed service, you are responsible for:
- Correct password hashing (bcrypt with cost factor ≥ 12, or argon2)
- Session management with rotation on privilege escalation
- CSRF protection for cookie-based auth
- OAuth state parameter validation
- Token storage (HttpOnly cookies only)
- Account enumeration prevention (see [backend/error-messages.md](../../backend/runtime/error-messages.md))
- Rate limiting on auth endpoints
- Session invalidation on logout and password change
- Secure password reset flow (time-limited, single-use tokens)

This is a significant surface area. Each item is a known failure mode with documented exploits.

## Related Documents

- [Session management](session-management.md)
- [Multi-factor authentication](mfa.md)
- [OAuth](oauth.md)
- [RBAC](rbac.md)
- [Backend error messages](../../backend/runtime/error-messages.md)
- [Auth pages layout](../../experience/layouts/auth-pages.md)
