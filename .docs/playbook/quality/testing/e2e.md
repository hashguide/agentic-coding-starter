---
kind: playbook
domain: quality
topics: [quality, testing, e2e]
authority: guidance
---

# End-to-End Testing

E2E tests exercise complete user flows through a real browser. They are the most expensive tests to write and maintain, but provide the highest confidence that the product works as a user experiences it.

## Playwright vs. Cypress

| Factor | Playwright | Cypress |
|---|---|---|
| **Browser support** | Chromium, Firefox, WebKit (Safari) | Chromium, Firefox, Electron |
| **Architecture** | Out-of-process, more stable | In-process, easier debugging |
| **Async handling** | Native async/await | Implicit command chaining |
| **CI performance** | Faster (parallel by default) | Good |
| **Cross-browser testing** | Better (WebKit/Safari) | Limited |
| **DX** | Excellent, growing | Mature, well-documented |

**Recommended:** Playwright for new projects. Cross-browser support (including Safari) is a significant advantage for web products. Cypress is equally valid if the team has existing expertise.

## Minimum Required E2E Flow Coverage

Write at least one E2E test for each of these flows:

| Flow | Why it's required |
|---|---|
| **Signup → onboarding → core action** | The most important user journey; must always work |
| **Upgrade to paid plan** | Revenue-critical; test with Stripe test mode |
| **Login with expired session** | Verify graceful redirect to login, not broken state |
| **Trigger and read a notification** | Verify the notification system end-to-end |
| **Initiate data export** | High-stakes user action; must complete without errors |
| **Complete account deletion** | Irreversible; must work correctly and confirm to user |

## Example Playwright Test

```typescript
// tests/e2e/signup.spec.ts
import { test, expect } from '@playwright/test';

test('new user can sign up and reach dashboard', async ({ page }) => {
  await page.goto('/');

  // Click the primary CTA
  await page.getByRole('link', { name: /get started/i }).click();
  await expect(page).toHaveURL('/signup');

  // Fill out signup form
  await page.getByLabel('Email').fill('new-user@example.com');
  await page.getByLabel('Password').fill('secure-password-123');
  await page.getByRole('button', { name: /create account/i }).click();

  // Should redirect to onboarding or dashboard
  await expect(page).toHaveURL(/\/(onboarding|dashboard)/);

  // Verify the user is logged in
  await expect(page.getByTestId('user-menu')).toBeVisible();
});

test('login with wrong password shows auth error', async ({ page }) => {
  await page.goto('/login');
  await page.getByLabel('Email').fill('existing@example.com');
  await page.getByLabel('Password').fill('wrong-password');
  await page.getByRole('button', { name: /sign in/i }).click();

  // Must show the standard auth error message — not "wrong password" or "user not found"
  await expect(page.getByRole('alert')).toContainText('Incorrect email or password');
});
```

## Playwright Configuration

```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  use: {
    baseURL: process.env.E2E_BASE_URL ?? 'http://localhost:3000',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    { name: 'chromium', use: { browserName: 'chromium' } },
    { name: 'webkit', use: { browserName: 'webkit' } },  // Safari
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## Running E2E Tests

```bash
# Run all E2E tests
npx playwright test

# Run a specific test file
npx playwright test tests/e2e/signup.spec.ts

# Run with headed browser (for debugging)
npx playwright test --headed

# Open Playwright's trace viewer after a failure
npx playwright show-trace trace.zip
```

## Related Documents

- [Testing strategy](strategy.md)
- [Testing error scenarios](error-scenarios.md)
