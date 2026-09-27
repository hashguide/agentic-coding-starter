---
kind: playbook
domain: architecture
topics: [architecture, patterns, creational, prototype]
authority: guidance
---

# Prototype Pattern

The Prototype pattern creates new objects by cloning an existing object (the prototype) rather than constructing from scratch. It is useful when object creation is expensive, when the exact type of object to create is not known until runtime, or when you want to create variations of a base configuration without repeating all its properties.

## Definition

A prototype is an object that can produce a copy of itself. The copy starts with the same state as the original and can be modified independently.

## JavaScript/TypeScript Relevance

JavaScript's object model is prototype-based, so cloning is built into the language. The pattern manifests in a few specific ways:

**Shallow clone — object spread:**
```typescript
const baseConfig = {
  timeout: 5000,
  retries: 3,
  baseUrl: 'https://api.example.com',
  headers: { 'Content-Type': 'application/json' },
};

// Shallow clone — headers object is still shared
const adminConfig = { ...baseConfig, timeout: 10_000 };
```

Spread creates a shallow clone: primitive values are copied, but nested objects and arrays are still shared references. Modifying `adminConfig.headers` would also modify `baseConfig.headers`.

**Deep clone — structuredClone:**
```typescript
// Deep clone — headers object is a new, independent copy
const adminConfig = structuredClone(baseConfig);
adminConfig.timeout = 10_000;
adminConfig.headers['X-Admin-Token'] = process.env.ADMIN_TOKEN!;
```

`structuredClone` (available in Node.js 17+ and modern browsers) is the correct tool for deep cloning plain objects. It handles nested objects, arrays, Date instances, Maps, and Sets correctly.

## When This Pattern Matters in SaaS Code

**Cloning complex configuration templates:** When building requests to an external API that share a base configuration but need per-request variations (auth headers, timeouts, specific params):

```typescript
const baseRequestConfig = {
  headers: {
    'Authorization': `Bearer ${process.env.API_KEY}`,
    'Content-Type': 'application/json',
  },
  timeout: 8000,
};

function makeRequest(endpoint: string, overrides: Partial<typeof baseRequestConfig>) {
  const config = structuredClone(baseRequestConfig);
  Object.assign(config.headers, overrides.headers);
  return fetch(endpoint, { ...config, ...overrides });
}
```

**Test fixture factories:** Cloning a base fixture and overriding specific fields is a prototype pattern in testing:

```typescript
const baseUser = {
  id: 'user-1',
  email: 'test@example.com',
  role: 'member' as const,
  subscription: 'free' as const,
};

const adminUser = { ...baseUser, id: 'admin-1', role: 'admin' as const };
const paidUser = { ...baseUser, id: 'paid-1', subscription: 'pro' as const };
```

## When This Pattern Is Rarely Needed

Most TypeScript SaaS code constructs objects through constructors, factory functions, or plain object literals. Explicit prototype-based cloning is rarely needed because:
- Most objects are small enough to construct directly
- ORMs return fresh instances on every query
- The performance benefit of cloning over construction only matters for objects that are genuinely expensive to create

Reach for `structuredClone` or spread when you find yourself constructing an object that is mostly identical to another with small differences, not as a general pattern.

## Related Documents

- [Factory pattern](factory.md)
- [Builder pattern](builder.md)
- [Patterns overview](../overview.md)
