---
kind: playbook
domain: analytics
topics: [identity, sessions, unique-views, pageviews, deduplication, analytics, unique, views]
authority: guidance
---
# Identity, Sessions, and Unique Views

Define the unit being counted before building analytics.

## Page/event count
Counts every qualifying occurrence. Useful for volume and workload, not audience size.

## Session
Groups activity into a visit according to an explicit inactivity/window rule. Useful for visit behavior and funnels.

## Unique user
Counts distinct identified users for a stated time window. Define anonymous-to-authenticated identity merging where needed.

## Unique resource view
Counts a person/resource combination once within a defined window, such as one product profile view per user/session/day. Specify anonymous identity, bots, owner views, repeated refreshes, privacy and deduplication.

Record the exact semantic definition beside the metric so dashboards cannot silently reinterpret it.
