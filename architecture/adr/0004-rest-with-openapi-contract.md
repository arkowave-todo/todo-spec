# ADR 0004: REST with an OpenAPI contract, owned by `todo-spec`

## Status
Proposed

## Context
Both frontends use the same API (section 3.2, I-1). The API contract is the only link between the frontends and the API (section 3.2 rules). system.md section 2.2 states the reason: "Teams work in parallel against one contract".

## Options considered
- REST with an OpenAPI contract, owned by `todo-spec` (chosen)
- Other options: Not recorded in system.md

## Decision
I-1 is a REST API described in OpenAPI. The contract lives in `todo-spec`, which owns it.

Reason: teams work in parallel against one contract.

## Consequences
- Stated: teams can work in parallel against one contract.
- Stated: the contract is the only link between the frontends and the API.
- Stated: the format of ids is set in the contract (D-1 rules).
- Inferred: a change to the API starts in `todo-spec`, then the code repos follow it. Code repos pin to a release tag (section 5.1).
- Inferred: the architect has to keep the contract in step with the requirements.

## Affects
S-1, S-2, S-3, C-1, C-2, C-4, I-1

## Related
ADR 0002, ADR 0006, ADR 0007, D-1
