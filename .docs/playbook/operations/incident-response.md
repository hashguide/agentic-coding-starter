---
kind: playbook
domain: operations
topics: [incident, outage, response, rollback, communication, operations]
authority: guidance
---
# Incident Response

Prepare a lightweight incident process before a production failure requires one.

1. Detect and confirm impact.
2. Establish an incident owner for coordination.
3. Stabilize the system: rollback, disable a feature, shed load, or switch to degraded behavior.
4. Preserve evidence and timestamps.
5. Communicate user impact through the appropriate channel.
6. Restore service and verify data integrity.
7. Write a concise follow-up: trigger, contributing conditions, detection gap, corrective actions and owners.

Runbooks should exist for the failures most likely to require urgent action, such as database recovery, payment/webhook failure, queue backlog, credential compromise or broken deployment.
