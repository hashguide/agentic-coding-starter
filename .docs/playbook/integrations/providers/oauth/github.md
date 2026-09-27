---
kind: playbook
domain: integrations
topics: [integrations, providers, oauth, github]
authority: guidance
---

# GitHub OAuth Setup

GitHub OAuth lets users sign in with their GitHub account. It's particularly useful for developer-focused products. GitHub does not include the user's email in the basic user profile response — a separate API call is required.

## Step 1 — Create a GitHub OAuth App

1. Go to **github.com** → Click your profile picture → **Settings**
2. In the left sidebar: **Developer settings** → **OAuth Apps**
3. Click **New OAuth App**
4. Fill in:
   - **Application name** — your product name
   - **Homepage URL** — `https://yourdomain.com`
   - **Authorization callback URL** — `https://yourdomain.com/auth/github/callback`
5. Click **Register application**

For local development, create a **separate** OAuth App with the callback URL set to `http://localhost:3000/auth/github/callback`. Do not share credentials between development and production.

## Step 2 — Get Credentials

1. On the app settings page, copy the **Client ID**
2. Click **Generate a new client secret**
3. Copy the secret immediately — it is only shown once

## Step 3 — Environment Variables

```
GITHUB_CLIENT_ID=Ov23li...
GITHUB_CLIENT_SECRET=...
```

## Step 4 — Implement the Callback Handler

GitHub's OAuth flow uses `user:email` scope to access email addresses. The primary verified email is not always in the basic user profile — it requires a call to `/user/emails`.

```typescript
import axios from 'axios';

// Redirect handler
app.get('/auth/github', (req, res) => {
  const state = randomBytes(32).toString('hex');
  req.session.oauthState = state;

  const params = new URLSearchParams({
    client_id: process.env.GITHUB_CLIENT_ID!,
    redirect_uri: `${process.env.APP_URL}/auth/github/callback`,
    scope: 'read:user user:email',
    state,
  });

  res.redirect(`https://github.com/login/oauth/authorize?${params}`);
});

// Callback handler
app.get('/auth/github/callback', async (req, res) => {
  const { code, state } = req.query as { code: string; state: string };

  if (state !== req.session.oauthState) {
    return res.status(400).send('Invalid state');
  }

  // Exchange code for access token
  const tokenRes = await axios.post(
    'https://github.com/login/oauth/access_token',
    {
      client_id: process.env.GITHUB_CLIENT_ID!,
      client_secret: process.env.GITHUB_CLIENT_SECRET!,
      code,
    },
    { headers: { Accept: 'application/json' } }
  );

  const accessToken = tokenRes.data.access_token;
  const authHeader = { Authorization: `Bearer ${accessToken}` };

  // Fetch user profile
  const [userRes, emailsRes] = await Promise.all([
    axios.get('https://api.github.com/user', { headers: authHeader }),
    axios.get('https://api.github.com/user/emails', { headers: authHeader }),
  ]);

  const githubUserId = String(userRes.data.id);
  const name: string = userRes.data.name ?? userRes.data.login;
  const avatarUrl: string = userRes.data.avatar_url;

  // Primary verified email — not always in profile, must fetch /user/emails
  const emails: Array<{ email: string; primary: boolean; verified: boolean }> = emailsRes.data;
  const primaryEmail = emails.find((e) => e.primary && e.verified)?.email;

  if (!primaryEmail) {
    return res.status(400).send('No verified email found on GitHub account');
  }

  // Find or create user
  let account = await db.oauthAccount.findUnique({
    where: { provider_providerUserId: { provider: 'github', providerUserId: githubUserId } },
    include: { user: true },
  });

  if (!account) {
    const user = await db.user.create({
      data: {
        email: primaryEmail,
        name,
        avatarUrl,
        oauthAccounts: {
          create: { provider: 'github', providerUserId: githubUserId },
        },
      },
    });
  }

  account = await db.oauthAccount.findUnique({
    where: { provider_providerUserId: { provider: 'github', providerUserId: githubUserId } },
    include: { user: true },
  });

  req.session.userId = account!.user.id;
  res.redirect('/dashboard');
});
```

## Scope Selection

| Scope | What it gives you |
|---|---|
| `read:user` | Basic public profile (id, name, avatar, public email) |
| `user:email` | All email addresses including private — required to get primary verified email |
| `read:org` | User's organization memberships (only if org gating is a core feature) |
| `repo` | Full repo read/write — only for GitHub-integrated features |

Request only what you need. `user:email` is always required to reliably retrieve a verified email.

## Organization Access Considerations

If your product gates access to members of a specific GitHub organization:

1. Request `read:org` scope
2. After authentication, call `GET https://api.github.com/orgs/{org}/members/{username}` to verify membership
3. Fall back gracefully if the user hasn't authorized your app to see their org membership

Be aware that org membership can change — do not cache it indefinitely. Check at login, not at every request.

## Related Documents

- [OAuth overview](overview.md)
- [Google OAuth setup](google.md)
- [OAuth testing](testing.md)
- [Auth options](../../../access/authentication/options.md)
