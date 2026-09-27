---
kind: playbook
domain: data
topics: [data, choosing]
authority: guidance
---

# Choosing a Database

The database type decision shapes every subsequent data access pattern. Most SaaS products should use a relational database — the question is only which one. Non-relational databases solve specific problems and are the wrong answer to most problems.

## Decision Table

| Database type | Data shape | Relationship complexity | Query patterns | Scale characteristics |
|---|---|---|---|---|
| **Relational (Postgres)** | Structured, tabular | High — joins, foreign keys, constraints | Flexible — SQL covers most patterns | Vertical + read replicas; handles most SaaS scale |
| **Document (MongoDB)** | Variable-shape documents | Low — embedded documents, references | Fast for denormalized reads; complex for relational queries | Horizontal sharding |
| **Graph (Neo4j)** | Nodes and edges | Extremely high — relationship traversal | Graph traversal; poor for tabular queries | Vertical, limited horizontal |
| **Time-series (TimescaleDB, InfluxDB)** | Ordered time-stamped events | Low | Time range queries, aggregations | Optimized for append-only time data |
| **Key-Value (Redis)** | Simple key → value | None | O(1) lookup by key | Horizontal; in-memory |

## The Case for PostgreSQL

PostgreSQL is the correct default for most SaaS products. It handles:
- Relational data with foreign keys, joins, and constraints
- JSON/JSONB columns when document-like flexibility is needed within a relational model
- Full-text search (for many use cases, eliminating the need for a separate search engine)
- Time-series data with TimescaleDB extension
- Graph-adjacent queries with recursive CTEs

A Postgres database handles the core product data, the job queue (pg-boss), the session store, the audit log, and feature flags — without introducing operational complexity for multiple database types.

**Choose Postgres unless:**
- Your primary data shape is genuinely document-like with no meaningful relationships (use MongoDB — but this is rare in SaaS)
- Your primary workload is high-frequency time-series data at scale (use TimescaleDB, which is an extension of Postgres)
- You need sub-millisecond key-value lookups at very high volume (use Redis as a complement to Postgres, not a replacement)

## Managed vs. Self-Hosted

| Option | Operational burden | Cost | Best for |
|---|---|---|---|
| **Managed (Supabase, Neon, Railway Postgres, RDS)** | Very low — no DBA work | Higher per GB | Solo builders, teams without DB ops expertise |
| **Self-hosted on VPS** | Medium — backups, updates, monitoring | Lower per GB | Cost-sensitive at scale, existing ops capability |

For a solo builder: start with a managed Postgres provider. The cost difference is irrelevant at small scale and the operational burden reduction is significant.

## Multi-Database Patterns

When a second database type is needed alongside Postgres, the common pattern is:
- **Postgres** — primary application data
- **Redis** — session store, rate limiting counters, feature flag cache, job queue
- **S3/Blob storage** — file uploads, generated exports

These complement Postgres rather than replacing it.

## Related Documents

- [Database ORM](orm.md)
- [Database schema design](schema-design.md)
- [Backend caching](../backend/architecture/caching.md)
- [Infrastructure hosting options](../delivery/infrastructure/hosting-options.md)
