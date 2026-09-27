---
kind: playbook
domain: frontend
topics: [frontend, app, types, ssr, static]
authority: guidance
---

# SSR and Static Rendering Frameworks

Server-side rendering (SSR) and static site generation (SSG) frameworks — Next.js, Astro, Nuxt — pre-render HTML on the server or at build time, enabling excellent SEO, fast initial load, and good performance on slow connections. The right choice depends on whether the content requires personalization and how frequently it changes.

## When SSR Is Justified

SSR renders HTML at request time on the server. It is justified when content must be:
- **SEO-critical:** The page must be crawled and indexed with real content (not a loading spinner)
- **Personalized:** The rendered HTML depends on who is requesting it (authenticated user's name, their account data)
- **Dynamic:** Content changes frequently enough that SSG's build-time generation would be stale

**Example use cases:** User profile pages (SEO + personalization), product pages in a marketplace (SEO + per-product data), news feeds.

## When Static Is Sufficient

Static generation renders HTML at build time and deploys as files. It is sufficient when content:
- **Is not personalized:** The same HTML is served to every visitor
- **Changes rarely:** A rebuild + redeploy is a reasonable update mechanism
- **Is SEO-critical:** SEO is excellent — the page is just a file

**Example use cases:** Marketing landing pages, pricing pages, documentation, blog posts.

## Framework Comparison

| Framework | Primary use | Rendering modes | Language ecosystem |
|---|---|---|---|
| **Next.js** | Full-stack React apps (SSR + SSG + CSR mixed) | All modes in one project | TypeScript/React |
| **Astro** | Content-heavy sites with minimal JS | SSG primary, SSR supported | Framework-agnostic; ship zero JS by default |
| **Nuxt** | Full-stack Vue apps | Same as Next.js but for Vue | TypeScript/Vue |
| **SvelteKit** | Full-stack Svelte apps | SSR + SSG + CSR | TypeScript/Svelte |

**One viable integrated approach:** a framework that supports static generation, server rendering, and client interactivity in the same application. Select the framework from the project stack, hosting model, rendering needs, and team proficiency.

## Mixing Strategies in Next.js

```typescript
// app/page.tsx — SSG (static, no dynamic data)
export default function LandingPage() {
  return <MarketingSite />;
}

// app/dashboard/page.tsx — CSR (client-rendered, personalized, no SEO)
'use client';
export default function Dashboard() {
  const { data } = useQuery({ queryKey: ['dashboard'], queryFn: fetchDashboard });
  return <DashboardUI data={data} />;
}

// app/blog/[slug]/page.tsx — SSG with ISR
export async function generateStaticParams() {
  const posts = await fetchAllPostSlugs();
  return posts.map((slug) => ({ slug }));
}

export const revalidate = 3600; // Revalidate every hour (ISR)

export default async function BlogPost({ params }: { params: { slug: string } }) {
  const post = await fetchPost(params.slug);
  return <BlogPostLayout post={post} />;
}
```

## Security

**Server-side secrets must never be included in the client bundle.** In Next.js, any variable used in a Server Component stays server-side. Variables used in Client Components (`'use client'`) are sent to the browser.

```typescript
// WRONG: secret exposed in client component
'use client';
const apiKey = process.env.MY_SECRET_KEY; // This is empty in the browser
                                            // but the intent was wrong

// CORRECT: secrets stay in Server Components
async function ServerComponent() {
  const data = await fetch('https://api.example.com', {
    headers: { Authorization: `Bearer ${process.env.MY_SECRET_KEY}` },
  });
  return <ClientComponent data={await data.json()} />;
}
```

**SSR injection vectors:** User-supplied content rendered server-side must be encoded before rendering. React handles this automatically for string interpolations; be careful with `dangerouslySetInnerHTML` and any raw HTML rendering.

Variables prefixed with `NEXT_PUBLIC_` in Next.js are explicitly included in the client bundle — never put secrets in `NEXT_PUBLIC_` variables.

## Related Documents

- [Frontend rendering strategies](../rendering.md)
- [Frontend performance](../performance.md)
- [SPA app type](spa.md)
