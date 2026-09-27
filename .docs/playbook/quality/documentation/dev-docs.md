---
kind: playbook
domain: quality
topics: [quality, documentation, dev, docs]
authority: guidance
---

# Developer Documentation

Developer documentation captures decisions, conventions, and operational procedures that are not obvious from reading the code. The goal is that any developer (including future-you) can get productive in hours, not days.

## Required Contents of a Project-Level CLAUDE.md

When using Claude Code on a project, a `CLAUDE.md` at the repo root provides session-persistent context:

```markdown
# Project Name

## What This Project Is
One paragraph: what the product does, who it's for, and what problem it solves.

## Tech Stack
- Runtime: Node.js 20 / TypeScript
- Framework: Express (backend), Next.js 14 (frontend)
- Database: PostgreSQL 16 via Prisma ORM
- Queue: BullMQ + Redis
- Hosting: Hetzner VPS, deployed via PM2

## Folder Structure
src/
  api/       → Express route handlers
  workers/   → Background job processors
  lib/       → Shared utilities (db, logger, mailer)
prisma/      → Schema and migrations
tests/       → unit/, integration/, e2e/

## Local Development
1. cp .env.example .env.local
2. docker-compose up -d  (starts Postgres and Redis)
3. npm run db:migrate
4. npm run db:seed
5. npm run dev

## Conventions
- All routes use Zod for request body validation
- All API errors return { error: ERROR_CODE_CONSTANT, referenceId: string }
- Background jobs are defined in src/workers/ and registered in src/workers/index.ts
- DB migrations are never destructive on first deploy — always additive, then backfill, then cleanup

## Anti-Patterns
- No raw process.env access outside of src/lib/env.ts
- No Prisma client instantiation outside of src/lib/db.ts
- No auth logic in route handlers — use requireAuth middleware

## Commands
- Tests: npm test
- Integration tests: npm run test:integration
- Migrations: npx prisma migrate dev
- Type check: npm run type-check
```

## README Template

The `README.md` is for external contributors and first-time reviewers:

```markdown
# Product Name

One-line description of what this does.

## Setup

Prerequisites: Node.js 20+, Docker (for Postgres + Redis)

\`\`\`bash
git clone <repo-url>
cd project
npm install
cp .env.example .env.local
docker-compose up -d
npm run db:migrate && npm run db:seed
npm run dev
\`\`\`

Open http://localhost:3000

## Running Tests

\`\`\`bash
npm test                    # unit tests
npm run test:integration    # integration tests (requires local DB)
\`\`\`

## Contributing

See CLAUDE.md for architecture conventions and anti-patterns.
```

## Architecture Decision Records (ADRs)

An ADR documents a decision, why it was made, and what alternatives were considered. Write one whenever a significant technical choice is made that isn't obvious from the code.

**ADR Template:**

```markdown
# ADR-NNN: Title

**Date:** 2025-01-15
**Status:** Accepted

## Context
What problem are we solving? What constraints exist?

## Decision
What did we decide to do?

## Alternatives Considered
- **Option A:** Description. Rejected because: reason.
- **Option B:** Description. Rejected because: reason.

## Consequences
What changes as a result of this decision? What new constraints does it introduce?
```

**Worked example:**

```markdown
# ADR-001: Use BullMQ over pg-boss for job queue

**Date:** 2025-01-10
**Status:** Accepted

## Context
We need a reliable background job queue for emails, exports, and account deletion. 
We already run Redis for session storage.

## Decision
Use BullMQ with the existing Redis instance.

## Alternatives Considered
- **pg-boss:** Uses Postgres, avoids Redis dependency. Rejected because we already have Redis 
  and pg-boss adds polling overhead to the database.
- **Inngest:** Managed, good DX. Rejected because vendor cost at scale and we want self-hosted.

## Consequences
Redis becomes a required dependency. BullMQ needs a separate worker process alongside the API server.
```

## Runbook Template

A runbook provides step-by-step instructions for an operational task:

```markdown
# Runbook: Production Deploy

## When to use this
When deploying code changes to production.

## Prerequisites
- CI is green on the branch to deploy
- Staging deployment has been verified
- No active incidents

## Steps
1. Ensure CI green: https://github.com/org/repo/actions
2. Merge PR to main
3. Wait for deploy pipeline to complete (~5 min)
4. Verify health endpoint: curl https://yourdomain.com/api/health
5. Check error monitoring for new errors: https://sentry.io/...
6. Monitor for 10 minutes

## Rollback
If the deploy causes errors:
1. Revert the PR on GitHub or push the previous commit to main
2. Wait for the deploy pipeline
3. Verify health endpoint

## Contact
On-call: ops@yourdomain.com
```

## OpenAPI Specification

Document your REST API with OpenAPI 3.x. Generate it from code when possible:

```typescript
// Using ts-rest or zod-to-openapi to generate OpenAPI from existing TypeScript types
import { generateOpenApi } from '@ts-rest/open-api';
import { contract } from './api/contract';

const openApiDocument = generateOpenApi(contract, {
  info: { title: 'Product API', version: '1.0.0' },
});
```

## Related Documents

- [Docs user documentation](user-docs.md)
- [Testing strategy](../testing/strategy.md)
