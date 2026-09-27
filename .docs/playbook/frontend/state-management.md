---
kind: playbook
domain: frontend
topics: [state, server-state, ui-state, url-state, form-state, global-state, frontend, management]
authority: guidance
---
# Frontend State Management

Classify state before choosing a library. Different kinds of state have different owners and lifetimes.

## Local UI state
Examples: disclosure state, temporary selection, menu visibility, local interaction state.

Keep it near the component or feature that owns it unless several distant surfaces genuinely coordinate through it.

## Server state
Examples: users, projects, jobs, notifications, billing state.

The server remains authoritative. Use the framework's server-data mechanisms or a server-state/cache library when caching, revalidation, retries, optimistic updates, pagination, and invalidation are meaningful.

## URL/navigation state
Examples: filters, sorting, selected tab/view, pagination, search terms.

Use the URL when state should survive refresh, support back/forward navigation, deep links, bookmarking, or sharing.

## Form/editing state
Keep draft values, validation, dirty state, autosave state, and submission state within the form/editor boundary. Choose explicit save, autosave, or optimistic editing from the interaction requirements.

## Shared client-only state
Examples: local theme preference, transient cross-surface selection, client-only workflow state.

Use framework facilities first; add a dedicated client store when shared update frequency, coordination, or ergonomics justify it.

## Persisted local state
Use browser/device storage for preferences, caches, offline data, or drafts only when lifecycle, versioning, privacy, synchronization, and conflict behavior are defined.

## Decision questions
- Who is the source of truth?
- Must this survive reload or be linkable?
- Which components need it?
- How can it become stale?
- Does it need optimistic updates or offline behavior?
- What invalidates it?

Record state ownership and synchronization rules for complex features in the feature technical design.
