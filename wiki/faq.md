# FAQ

## Should an agent read the whole playbook?
No. Use `ROUTING.md` or the applicable repository skill and load the smallest relevant set.

## Where do I put my own preferences?
`.docs/project/constraints.md`.

## What if the idea is extremely vague?
Start with the discovery prompt. The agent is expected to create the professional specification, not wait for the owner to enumerate every requirement.

## What needs owner approval?
Choices that materially alter product direction, scope, business model, important user experience, public contracts, legal/data guarantees, or expensive commitments. Routine implementation choices belong to the agent.

## Are all playbook topics mandatory?
No. The manifest should resolve each relevant domain as specified, deferred, or not applicable.

## How are docs kept from becoming stale?
The implementation loop requires project truth to be updated when behavior, contracts or architecture change. Review agents should also check documentation against the implemented behavior during meaningful validation passes.
