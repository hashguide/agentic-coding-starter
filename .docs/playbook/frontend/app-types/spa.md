---
kind: playbook
domain: frontend
topics: [frontend, app, types, spa]
authority: guidance
---

# Single Page Application (SPA)

A Single Page Application loads once and navigates between views without full page reloads. JavaScript handles routing, data fetching, and DOM updates. SPAs are the right choice for authenticated application UIs — dashboards, editors, data management tools — where SEO is not a concern and the user expects fast in-app navigation.

## Routing

Client-side routing intercepts link clicks and URL changes, rendering the appropriate component without a server round-trip.

```typescript
// React Router v6
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/',
    element: <AppShell />,
    children: [
      { index: true, element: <Dashboard /> },
      { path: 'invoices', element: <InvoiceList /> },
      { path: 'invoices/:id', element: <InvoiceDetail /> },
      { path: 'settings', element: <Settings /> },
    ],
  },
  { path: '/login', element: <LoginPage /> },
  { path: '/signup', element: <SignupPage /> },
]);

export function App() {
  return <RouterProvider router={router} />;
}
```

## Code Splitting Strategy

A SPA that bundles all routes into one JavaScript file defeats the purpose. Split at the route level so users only load what they navigate to.

```typescript
const InvoiceList = lazy(() => import('./features/invoices/InvoiceList'));
const InvoiceDetail = lazy(() => import('./features/invoices/InvoiceDetail'));
const Settings = lazy(() => import('./features/settings/Settings'));

const router = createBrowserRouter([
  {
    path: '/',
    element: <AppShell />,
    children: [
      { index: true, element: <Dashboard /> },
      {
        path: 'invoices',
        element: (
          <Suspense fallback={<PageSkeleton />}>
            <InvoiceList />
          </Suspense>
        ),
      },
    ],
  },
]);
```

## Application Shell Model

The application shell is the minimal HTML/CSS/JS needed to render the navigation structure immediately — before data loads. Data loads asynchronously and fills in the shell.

```
┌─────────────────────────────────────┐
│ NavBar (shell — loads immediately)  │
├──────────┬──────────────────────────┤
│ Sidebar  │  Content area            │
│ (shell)  │  ┌────────────────────┐  │
│          │  │ Data (loads async) │  │
│          │  │ Shows skeleton     │  │
│          │  │ while loading      │  │
│          │  └────────────────────┘  │
└──────────┴──────────────────────────┘
```

The shell renders instantly from the service worker cache or CDN; the content area shows skeleton loaders while data fetches.

## Security

**XSS surface area:** SPAs have a larger XSS attack surface than server-rendered apps. All user-controlled content rendered into the DOM must be encoded. React escapes string interpolations by default (`{userInput}`), but `dangerouslySetInnerHTML` bypasses this protection and must never be used with user-supplied content.

**Auth tokens must never be stored in localStorage.** localStorage is accessible to any JavaScript executing on the page. A successful XSS attack can exfiltrate localStorage contents, including auth tokens, with a single line of code. Auth tokens belong in HttpOnly cookies, which are inaccessible to JavaScript.

```typescript
// WRONG: auth token in localStorage
localStorage.setItem('token', jwt);

// CORRECT: token in HttpOnly cookie — set by the server, never readable by JS
// Server sets:
res.cookie('session', sessionId, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
});
// Client never reads the token directly — it's sent automatically by the browser
```

## Related Documents

- [Frontend rendering](../rendering.md)
- [Auth session management](../../access/authentication/session-management.md)
- [Client storage](client-storage.md)
- [Frontend performance](../performance.md)
