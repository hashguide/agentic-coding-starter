---
kind: playbook
domain: analytics
topics: [measurement, product-analytics, metrics]
authority: guidance
---
# Measurement Plan

Start with product questions and decisions, then define events and metrics needed to answer them.

## Define outcomes
Identify:
- acquisition or entry point
- activation: the first evidence the user received core value
- repeated value/retention behavior
- key workflow completion and failure
- monetization/upgrade behavior where applicable
- quality or reliability signals that affect product decisions

## Metric types
Use counts/rates for operational frequency, funnels for ordered journeys, cohorts for retention, distributions for latency/value ranges, and unique-user/resource metrics when repeated events would distort the question.

## Event design
For each event define name, trigger, actor identity, tenant/account context, relevant resource IDs, allowed properties, timestamp source, and whether the event is client- or server-authoritative. Avoid sensitive payloads that are not required for analysis.

## Identity
Define anonymous/session/user/account identity and merge rules before relying on unique-user or funnel analysis. Clarify what a unique view or active user means for the product.

## Quality
Plan for duplicates, retries, blocked clients, offline delivery, schema evolution, bot/internal traffic, and test environments. Critical business events may need server-side emission or reconciliation.

## Record
Maintain the product questions, event catalog, metric definitions, ownership, retention/privacy requirements, and dashboards/queries that support launch and iteration decisions.
