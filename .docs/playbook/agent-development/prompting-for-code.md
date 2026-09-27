---
kind: playbook
domain: agent-development
topics: [agent, development, prompting, for, code]
authority: guidance
---

# Prompting for Code Generation

The quality of generated code is bounded by the quality of the prompt. A vague prompt produces code that technically works for the imagined case while missing every relevant constraint. A specific prompt produces code that slots into the actual system. The difference is not the model — it's the framing.

## Structure of a Good Code-Generation Prompt

Every effective code-generation prompt contains four parts:

**1. Context** — where this code lives and what it connects to
> "I'm adding an API endpoint to a Next.js 14 app using tRPC and Prisma with a PostgreSQL database."

**2. Constraint** — what rules the code must follow (the most commonly omitted part)
> "All errors must return a structured error code, not raw error messages. No raw SQL — ORM only. PII must not appear in logs."

**3. Example** — a reference point for shape and style (optional, but cuts ambiguity dramatically)
> "Existing endpoints look like this: [paste 20 lines of an existing router]"

**4. Acceptance criteria** — what "done" means so the agent can self-check
> "The endpoint should: accept a userId and invoiceId, verify the user owns the invoice, return the invoice with its line items, return a NOT_FOUND error code if the invoice doesn't exist or doesn't belong to this user."

## Before/After Example

**Vague prompt:**
> "Write a function to send an email when a payment fails."

This produces a function that calls an email SDK with hardcoded strings, no error handling, no logging, no retry logic, and no regard for the email provider pattern this project uses.

**Specific prompt:**
> "I'm using the Resend SDK for email in a Node.js TypeScript backend. When a payment fails (triggered by a Stripe webhook), I need to send a transactional email to the customer. The email template ID is already configured — I just need the function that calls Resend with the customer's email address and the invoice amount. Follow the pattern in `src/lib/email.ts`. Errors from Resend should be caught and logged with the requestId field using our pino logger — never let the email error propagate up to the webhook handler, since the webhook must return 200 to Stripe regardless."

This prompt produces usable, correctly-constrained code on the first attempt.

## Referencing Playbook Documents

When the task touches an area covered by the playbook, reference the document explicitly in the prompt:

> "Before writing this, read `backend/error-messages.md` in the playbook. The error response format must follow that document exactly."

> "The webhook handler must follow the four-step pattern in `workers/webhooks.md`: return 200 immediately, verify the signature, enqueue, then process."

Agents with access to the playbook via file tools will read the document and apply its rules. Agents without access benefit from the reminder to ask or from you pasting the relevant section.

## Related Documents

- [Context management](context-management.md)
- [Reviewing AI code](reviewing-ai-code.md)
- [Prompt library](prompt-library.md)
- [Handling hallucination](handling-hallucination.md)
