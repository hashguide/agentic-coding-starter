---
kind: playbook
domain: integrations
topics: [integrations, providers, oauth, testing]
authority: guidance
---

# OAuth Testing

Testing OAuth flows requires handling the redirect-based authorization code exchange, which makes automated testing more involved than standard API calls.

## Local Development — Configure Real Redirect URIs

Both Google and GitHub allow `http://localhost` as a redirect URI for local development.

**Google:** In Google Cloud Console → Credentials → your OAuth client → add `http://localhost:3000/auth/google/callback` under Authorized redirect URIs.

**GitHub:** Create a separate development OAuth App with callback URL `http://localhost:3000/auth/github/callback`. Never reuse production credentials in development.

Run your app locally and test the full browser flow — clicking "Sign in with Google/GitHub" and completing the real OAuth consent screen. This is the best manual test because it exercises the full flow including state parameter and token exchange.

## Automated Tests — Mock the OAuth Provider

Do not call real provider APIs in unit or integration tests. The authorization code exchange cannot be automated without controlling the provider's redirect.

**Approach: mock the token exchange and profile endpoints**

```typescript
// src/__mocks__/oauthProviders.ts

// Mock the Google token verification
jest.mock('google-auth-library', () => ({
  OAuth2Client: jest.fn().mockImplementation(() => ({
    generateAuthUrl: jest.fn().mockReturnValue('https://mock.google.com/auth'),
    getToken: jest.fn().mockResolvedValue({
      tokens: { id_token: 'mock_id_token', access_token: 'mock_access_token' },
    }),
    verifyIdToken: jest.fn().mockResolvedValue({
      getPayload: () => ({
        sub: 'google_user_123',
        email: 'test@example.com',
        name: 'Test User',
        picture: 'https://example.com/avatar.jpg',
      }),
    }),
  })),
}));

// Mock axios for GitHub token exchange
jest.mock('axios');
const mockedAxios = axios as jest.Mocked<typeof axios>;

mockedAxios.post.mockResolvedValue({ data: { access_token: 'mock_github_token' } });
mockedAxios.get
  .mockResolvedValueOnce({
    data: { id: 42, name: 'Test User', login: 'testuser', avatar_url: 'https://...' },
  })
  .mockResolvedValueOnce({
    data: [{ email: 'test@example.com', primary: true, verified: true }],
  });
```

```typescript
// Integration test — simulate receiving the callback with a valid state
test('GitHub callback creates a new user and session', async () => {
  const agent = supertest.agent(app); // agent persists cookies/session

  // Prime the session with a valid OAuth state
  await agent.get('/auth/github').expect(302);

  // Get the state that was stored in session
  const state = 'mock_state';
  // Directly set the state in test session if your session allows it,
  // or extract it from the redirect URL in the test

  const res = await agent
    .get(`/auth/github/callback?code=test_code&state=${state}`)
    .expect(302);

  expect(res.headers.location).toBe('/dashboard');

  const user = await db.user.findUnique({ where: { email: 'test@example.com' } });
  expect(user).toBeDefined();
  expect(user?.name).toBe('Test User');
});
```

## Testing State Parameter Validation

Always assert that a mismatched or missing state parameter is rejected:

```typescript
test('GitHub callback rejects invalid state', async () => {
  const agent = supertest.agent(app);

  // Start the flow to create a valid session state
  await agent.get('/auth/github');

  // Submit callback with wrong state
  const res = await agent
    .get('/auth/github/callback?code=test_code&state=invalid_state')
    .expect(400);
});

test('GitHub callback rejects missing state', async () => {
  const agent = supertest.agent(app);
  await agent.get('/auth/github');

  await agent.get('/auth/github/callback?code=test_code').expect(400);
});
```

## Integration Test Checklist

Before merging OAuth changes:

- [ ] New user sign-in creates a user record with correct email and provider ID
- [ ] Returning user sign-in finds the existing account (not creates a duplicate)
- [ ] Invalid state returns 400 (CSRF protection works)
- [ ] Missing email (GitHub: no verified email) is handled gracefully
- [ ] Manual end-to-end browser test completed for both providers in local dev

## Related Documents

- [OAuth overview](overview.md)
- [Google OAuth setup](google.md)
- [GitHub OAuth setup](github.md)
- [Auth options](../../../access/authentication/options.md)
