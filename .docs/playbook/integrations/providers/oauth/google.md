---
kind: playbook
domain: integrations
topics: [integrations, providers, oauth, google]
authority: guidance
---

# Google OAuth Setup

Google OAuth lets users sign in with their Google account. Google's consent screen and verification requirements add more initial setup than most providers.

## Step 1 — Create a Google Cloud Project

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Click **Select a project** → **New Project**
3. Name the project (use your product name)
4. Click **Create**

## Step 2 — Enable the OAuth API

1. In the left sidebar: **APIs & Services** → **Enabled APIs & Services**
2. Click **+ Enable APIs and Services**
3. Search for and enable **Google+ API** (or **Google Identity** / **People API** if needed for profile data)

## Step 3 — Configure the OAuth Consent Screen

1. Navigate to **APIs & Services** → **OAuth consent screen**
2. Select **External** (for a public-facing app) or **Internal** (G Suite / Workspace org only)
3. Fill in required fields:
   - **App name** — your product name
   - **User support email** — your support email address
   - **Developer contact information** — your email
4. On the **Scopes** screen, add only the scopes you need:
   - `openid` — required for OIDC authentication
   - `email` — user's email address
   - `profile` — user's name and avatar
5. On **Test users**, add your own email to test while the app is in "Testing" status

The app remains in "Testing" status (max 100 users) until you submit it for Google's verification review. For internal tools or controlled user groups, Testing status is often sufficient indefinitely.

## Step 4 — Create OAuth Credentials

1. Navigate to **APIs & Services** → **Credentials**
2. Click **+ Create Credentials** → **OAuth client ID**
3. Application type: **Web application**
4. Add **Authorized redirect URIs**:
   - `http://localhost:3000/auth/google/callback` (development)
   - `https://yourdomain.com/auth/google/callback` (production)
5. Click **Create**
6. Copy the **Client ID** and **Client Secret**

## Step 5 — Environment Variables

```
GOOGLE_CLIENT_ID=<your-client-id>.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=GOCSPX-...
```

## Step 6 — Implement the Callback Handler

```typescript
import { OAuth2Client } from 'google-auth-library';

const googleClient = new OAuth2Client({
  clientId: process.env.GOOGLE_CLIENT_ID!,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
  redirectUri: `${process.env.APP_URL}/auth/google/callback`,
});

// Redirect handler — send user to Google
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

// Callback handler — receive authorization code
app.get('/auth/google/callback', async (req, res) => {
  const { code, state } = req.query as { code: string; state: string };

  if (state !== req.session.oauthState) {
    return res.status(400).send('Invalid state');
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
  const avatarUrl = payload.picture;

  // Find or create user
  let account = await db.oauthAccount.findUnique({
    where: { provider_providerUserId: { provider: 'google', providerUserId: googleUserId } },
    include: { user: true },
  });

  if (!account) {
    const user = await db.user.create({
      data: {
        email,
        name,
        avatarUrl,
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

## Recommended Scopes

| Scope | What it gives you | When to request |
|---|---|---|
| `openid` | Verifiable user identity (sub) | Always |
| `email` | User's email address | Always |
| `profile` | Name, picture URL | If displaying the user's name/avatar |
| `https://www.googleapis.com/auth/calendar` | Calendar read/write | Only if calendar is a core feature |

Request the minimum scopes needed. Broad scope requests trigger Google's extended verification review.

## Related Documents

- [OAuth overview](overview.md)
- [OAuth testing](testing.md)
- [Auth options](../../../access/authentication/options.md)
