---
kind: playbook
domain: access
topics: [access, authentication, oauth]
authority: guidance
---

# OAuth 2.0

OAuth 2.0 allows users to authenticate with your application using their existing accounts at identity providers (Google, GitHub, etc.) without sharing their passwords. The implementation has several non-obvious security requirements that are easy to omit.

## Authorization Code Flow

```
1. User clicks "Sign in with Google"
2. App redirects to Google's authorization URL with:
   - client_id (app identifier)
   - redirect_uri (where Google sends the user back)
   - scope (what permissions you're requesting)
   - state (random value for CSRF protection)
   - code_challenge (PKCE — for public clients)

3. User authenticates with Google and grants consent
4. Google redirects to redirect_uri with:
   - code (one-time authorization code)
   - state (must match what you sent in step 2)

5. App validates state parameter
6. App exchanges code for tokens:
   POST to Google token endpoint with code + client_secret

7. Google returns:
   - access_token (for API calls)
   - id_token (contains user identity claims)

8. App validates id_token signature
9. App extracts user identifier from id_token claims
10. App creates or retrieves local user, creates session
```

## Account Linking Problem

**Never join by email.** Multiple OAuth providers can have the same email address registered. An attacker who controls a provider account with email `victim@example.com` can sign into your app as any user with that email address if you join by email.

**Join by provider user ID:**

```typescript
async function findOrCreateUserFromOAuth(params: {
  provider: 'google' | 'github';
  providerUserId: string;
  email: string;
  name: string;
}): Promise<User> {
  // Look up by provider + provider user ID — not email
  const existing = await db.oauthAccount.findUnique({
    where: {
      provider_providerUserId: {
        provider: params.provider,
        providerUserId: params.providerUserId,
      },
    },
    include: { user: true },
  });

  if (existing) return existing.user;

  // New user — create user and link OAuth account
  const user = await db.$transaction(async (tx) => {
    const newUser = await tx.user.create({
      data: { email: params.email, name: params.name },
    });
    await tx.oauthAccount.create({
      data: {
        userId: newUser.id,
        provider: params.provider,
        providerUserId: params.providerUserId,
      },
    });
    return newUser;
  });

  return user;
}
```

## Security

**State parameter CSRF protection:** Generate a random value before redirecting to the provider, store it in the session, and verify it matches the `state` returned in the callback. Without this, an attacker can trick a user into completing an OAuth flow that links the attacker's provider account to the victim's app account.

```typescript
// On redirect: generate and store state
const state = crypto.randomBytes(16).toString('hex');
req.session.oauthState = state;
res.redirect(buildAuthorizationUrl({ state }));

// On callback: verify state matches
if (req.query.state !== req.session.oauthState) {
  return res.status(400).json({ code: 'OAUTH_STATE_MISMATCH' });
}
delete req.session.oauthState;
```

**PKCE for public clients:** Any client where the client_secret cannot be kept secret (mobile apps, SPAs without a backend) must use PKCE (Proof Key for Code Exchange). PKCE generates a code verifier and code challenge — the authorization code can only be exchanged by the holder of the original code verifier.

**Never use email as the join key.** Always use the provider's stable user identifier (`sub` in Google's id_token, `id` in GitHub's user object).

## Related Documents

- [Auth options](options.md)
- [OAuth integration overview](../../integrations/providers/oauth/overview.md)
- [OAuth Google setup](../../integrations/providers/oauth/google.md)
- [OAuth GitHub setup](../../integrations/providers/oauth/github.md)
