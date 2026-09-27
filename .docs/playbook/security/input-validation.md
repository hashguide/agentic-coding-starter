---
kind: playbook
domain: security
topics: [security, input, validation]
authority: guidance
---

# Input Validation

All user input is untrusted. Every value arriving from a request — body, query string, headers, path parameters — must be validated against an expected schema before it touches business logic. Validation at the boundary prevents entire classes of attack: injection, unexpected type coercions, and oversized payloads.

## Schema Validation at the Boundary

Validate at the entry point of every route, not inside business logic. Business logic should receive data that has already been verified to be the expected shape and type.

```typescript
import { z } from 'zod';
import { asyncHandler } from '../middleware/async-handler';
import { ValidationError } from '../errors';

const createProjectSchema = z.object({
  name: z.string().min(1).max(100).trim(),
  description: z.string().max(500).optional(),
  visibility: z.enum(['public', 'private']),
});

app.post('/api/projects', requireAuth, asyncHandler(async (req, res) => {
  const body = createProjectSchema.safeParse(req.body);
  if (!body.success) throw new ValidationError(body.error.flatten());

  const project = await projectService.create({
    ...body.data,
    userId: req.userId,
  });
  res.status(201).json(project);
}));
```

## XSS Prevention

XSS (cross-site scripting) attacks inject malicious scripts into pages viewed by other users. Prevention has two parts:

**1. Validate on input.** Reject or strip HTML from fields that should not contain it. A `name` field should never contain `<script>`.

**2. Encode on output.** Frameworks like React escape HTML by default — never use `dangerouslySetInnerHTML` with user-controlled content. In server-rendered templates, use the framework's auto-escaping; never use string concatenation to build HTML.

```tsx
// Safe: React escapes this automatically
function UserGreeting({ name }: { name: string }) {
  return <p>Hello, {name}</p>;
}

// Unsafe: never do this
function UserGreeting({ name }: { name: string }) {
  return <p dangerouslySetInnerHTML={{ __html: `Hello, ${name}` }} />;
}
```

If users submit rich text that must render as HTML, use a server-side sanitization library (DOMPurify via jsdom, or the `sanitize-html` package) with a strict allowlist of permitted tags and attributes.

## SQL Injection Prevention

Use the ORM for all database queries. The ORM parameterizes all values, making SQL injection structurally impossible for standard queries.

```typescript
// Safe: Prisma parameterizes userId automatically
const projects = await db.project.findMany({
  where: { userId: req.userId, deletedAt: null },
});

// Unsafe: raw string interpolation into SQL
const projects = await db.$queryRawUnsafe(
  `SELECT * FROM projects WHERE user_id = '${req.userId}'`
);
```

If you must write raw SQL (for performance or advanced queries), use parameterized raw queries only:

```typescript
// Safe: parameterized even in raw SQL
const result = await db.$queryRaw`
  SELECT * FROM projects WHERE user_id = ${req.userId}
`;
```

See [database/orm.md](../data/orm.md) for ORM selection guidance.

## File Upload Validation

File uploads require server-side validation — client-side validation is trivially bypassed.

```typescript
import { createReadStream } from 'fs';
import { lookup } from 'mime-types';

const ALLOWED_MIME_TYPES = ['image/jpeg', 'image/png', 'image/webp', 'image/gif'];
const MAX_FILE_SIZE_BYTES = 5 * 1024 * 1024; // 5 MB

function validateUpload(file: Express.Multer.File): void {
  // Size limit enforced at upload middleware level (before reaching this function)
  if (file.size > MAX_FILE_SIZE_BYTES) {
    throw new ValidationError({ file: ['File exceeds the 5 MB size limit'] });
  }

  // Server-side MIME type check — never trust the Content-Type header alone
  // Use the file's magic bytes (first few bytes) via a library like 'file-type'
  const detectedType = lookup(file.originalname) || 'application/octet-stream';
  if (!ALLOWED_MIME_TYPES.includes(detectedType)) {
    throw new ValidationError({ file: ['File type not allowed'] });
  }
}
```

**Checklist for file uploads:**

- [ ] Size limit enforced at the web server / middleware layer (before processing)
- [ ] MIME type verified from file magic bytes, not the `Content-Type` header or file extension
- [ ] Files stored outside the web root (not publicly accessible by default)
- [ ] File names sanitized before storage — never use the original filename as a storage path
- [ ] Consider malware scanning (ClamAV, VirusTotal API) for files that will be shared with other users

## Related Documents

- [Security overview](overview.md)
- [Database ORM](../data/orm.md)
- [Backend error handling](../backend/runtime/error-handling.md)
- [Security checklist](checklist.md)
