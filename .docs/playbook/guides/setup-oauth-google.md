---
kind: playbook
domain: guides
topics: [guides, setup, oauth, google]
authority: guidance
---

# How to Setup Google OAuth

**What you'll have at the end:** Working Google sign-in with user profile retrieval and account creation.

---

## Step 1 — Create a Google Cloud Project

1. Go to console.cloud.google.com
2. Click **Select a project → New Project**
3. Name the project (use your product name)
4. Click **Create**

---

## Step 2 — Configure the OAuth Consent Screen

1. Left sidebar → **APIs & Services → OAuth consent screen**
2. Select **External**
3. Fill in: App name, support email, developer contact email
4. On the Scopes page, click **Add or Remove Scopes** and add:
   - `openid`
   - `https://www.googleapis.com/auth/userinfo.email`
   - `https://www.googleapis.com/auth/userinfo.profile`
5. On Test Users, add your email address
6. Save and continue

---

## Step 3 — Create OAuth Credentials

1. Left sidebar → **APIs & Services → Credentials**
2. **+ Create Credentials → OAuth client ID**
3. Application type: **Web application**
4. Authorized redirect URIs:
   - `http://localhost:3000/auth/google/callback` (dev)
   - `https://yourdomain.com/auth/google/callback` (production)
5. Click **Create** and copy the Client ID and Client Secret

```
GOOGLE_CLIENT_ID=<client-id>.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=GOCSPX-...
```

---

## Step 4 — Install the Google Auth Library

```bash
npm install google-auth-library
```

---

## Step 5 — Implement the Redirect Route

```typescript
import { OAuth2Client } from 'google-auth-library';
import { randomBytes } from 'crypto';

const googleClient = new OAuth2Client({
  clientId: process.env.GOOGLE_CLIENT_ID!,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
  redirectUri: `${process.env.APP_URL}/auth/google/callback`,
});

app.get('/auth/google', (req, res) => {
  const state = randomBytes(32).toString('hex');
  req.session.oauthState = state;

  const url = googleClient.generateAuthUrl({
    access_type: 'offline',
    scope: ['openid', 'email', 'profile'],
    state,
  });

  res.redirect(url);
});
```

---

## Step 6 — Implement the Callback Route

```typescript
app.get('/auth/google/callback', async (req, res) => {
  const { code, state } = req.query as { code: string; state: string };

  if (state !== req.session.oauthState) {
    return res.status(400).send('Invalid state parameter');
  }

  const { tokens } = await googleClient.getToken(code);
  const ticket = await googleClient.verifyIdToken({
    idToken: tokens.id_token!,
    audience: process.env.GOOGLE_CLIENT_ID!,
  });

  const payload = ticket.getPayload()!;
  const googleUserId = payload.sub;
  const email = payload.email!;
  const name = payload.name ?? email;

  // Find or create user account
  let account = await db.oauthAccount.findUnique({
    where: { provider_providerUserId: { provider: 'google', providerUserId: googleUserId } },
    include: { user: true },
  });

  if (!account) {
    await db.user.create({
      data: {
        email,
        name,
        avatarUrl: payload.picture,
        oauthAccounts: {
          create: { provider: 'google', providerUserId: googleUserId },
        },
      },
    });
    account = await db.oauthAccount.findUnique({
      where: { provider_providerUserId: { provider: 'google', providerUserId: googleUserId } },
      include: { user: true },
    });
  }

  req.session.userId = account!.user.id;
  res.redirect('/dashboard');
});
```

---

## Step 7 — Add the Button to Your Signin Page

```tsx
<a href="/auth/google">
  <button>Continue with Google</button>
</a>
```

---

## Step 8 — Test End-to-End

1. Start the dev server
2. Click "Continue with Google"
3. Sign in with your Google test user account
4. Verify redirect to `/dashboard`
5. Check the database for the created user and oauth_account records

---

## Verify

- [ ] Redirect to Google works
- [ ] Callback creates a user record on first login
- [ ] Callback signs in existing user on subsequent logins (no duplicate users)
- [ ] Invalid state parameter returns 400
- [ ] Production redirect URI registered in Google Cloud Console

---

## Related Documents

- [Integrations OAuth overview](../integrations/providers/oauth/overview.md)
- [Integrations OAuth Google](../integrations/providers/oauth/google.md)
- [Integrations OAuth testing](../integrations/providers/oauth/testing.md)
