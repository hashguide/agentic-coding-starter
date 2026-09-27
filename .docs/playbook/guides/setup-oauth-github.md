---
kind: playbook
domain: guides
topics: [guides, setup, oauth, github]
authority: guidance
---

# How to Setup GitHub OAuth

**What you'll have at the end:** Working GitHub sign-in with verified email retrieval and account creation.

---

## Step 1 — Create a GitHub OAuth App

1. github.com → Profile picture → **Settings**
2. Left sidebar → **Developer settings → OAuth Apps**
3. Click **New OAuth App**
4. Fill in:
   - **Application name:** your product name
   - **Homepage URL:** `https://yourdomain.com`
   - **Authorization callback URL:** `https://yourdomain.com/auth/github/callback`
5. Click **Register application**

Create a **separate app** for local development with callback URL `http://localhost:3000/auth/github/callback`. Never share credentials between environments.

---

## Step 2 — Get Credentials

1. On the app page, copy the **Client ID**
2. Click **Generate a new client secret**
3. Copy the secret immediately

```
GITHUB_CLIENT_ID=Ov23li...
GITHUB_CLIENT_SECRET=...
```

---

## Step 3 — Install Dependencies

```bash
npm install axios
```

---

## Step 4 — Implement the Redirect Route

```typescript
import { randomBytes } from 'crypto';

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
```

---

## Step 5 — Implement the Callback Route

```typescript
import axios from 'axios';

app.get('/auth/github/callback', async (req, res) => {
  const { code, state } = req.query as { code: string; state: string };

  if (state !== req.session.oauthState) {
    return res.status(400).send('Invalid state parameter');
  }

  // Exchange code for access token
  const tokenRes = await axios.post(
    'https://github.com/login/oauth/access_token',
    { client_id: process.env.GITHUB_CLIENT_ID!, client_secret: process.env.GITHUB_CLIENT_SECRET!, code },
    { headers: { Accept: 'application/json' } }
  );
  const accessToken: string = tokenRes.data.access_token;
  const authHeader = { Authorization: `Bearer ${accessToken}` };

  // Fetch user profile and emails (email is not always in profile)
  const [userRes, emailsRes] = await Promise.all([
    axios.get('https://api.github.com/user', { headers: authHeader }),
    axios.get('https://api.github.com/user/emails', { headers: authHeader }),
  ]);

  const githubUserId = String(userRes.data.id);
  const name: string = userRes.data.name ?? userRes.data.login;

  const emails: Array<{ email: string; primary: boolean; verified: boolean }> = emailsRes.data;
  const primaryEmail = emails.find((e) => e.primary && e.verified)?.email;

  if (!primaryEmail) {
    return res.status(400).send('No verified primary email found on GitHub account');
  }

  // Find or create user
  let account = await db.oauthAccount.findUnique({
    where: { provider_providerUserId: { provider: 'github', providerUserId: githubUserId } },
    include: { user: true },
  });

  if (!account) {
    await db.user.create({
      data: {
        email: primaryEmail,
        name,
        avatarUrl: userRes.data.avatar_url,
        oauthAccounts: {
          create: { provider: 'github', providerUserId: githubUserId },
        },
      },
    });
    account = await db.oauthAccount.findUnique({
      where: { provider_providerUserId: { provider: 'github', providerUserId: githubUserId } },
      include: { user: true },
    });
  }

  req.session.userId = account!.user.id;
  res.redirect('/dashboard');
});
```

---

## Step 6 — Add the Button to Your Signin Page

```tsx
<a href="/auth/github">
  <button>Continue with GitHub</button>
</a>
```

---

## Step 7 — Test End-to-End

1. Start the dev server (with development OAuth App credentials)
2. Click "Continue with GitHub"
3. Authorize the app on GitHub
4. Verify redirect to `/dashboard`
5. Check the database for the created user record

---

## Verify

- [ ] Redirect to GitHub works
- [ ] Callback creates a user on first login
- [ ] Callback finds existing user on subsequent logins (no duplicate records)
- [ ] Invalid state returns 400
- [ ] App handles accounts with no verified email gracefully
- [ ] Production OAuth App has the production callback URL

---

## Related Documents

- [Integrations OAuth overview](../integrations/providers/oauth/overview.md)
- [Integrations OAuth GitHub](../integrations/providers/oauth/github.md)
- [Integrations OAuth testing](../integrations/providers/oauth/testing.md)
