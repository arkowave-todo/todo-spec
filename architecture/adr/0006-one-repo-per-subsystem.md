# ADR 0006: One repo per subsystem

## Status
Proposed

## Context
The system has three subsystems (section 3.1). Each one is owned by a team and has one repo: S-1 `todo-fe-web`, S-2 `todo-api`, S-3 `todo-fe-ios`. system.md section 2.2 states the reason: "Team and permission boundaries".

## Options considered
- One repo per subsystem (chosen)
- Other options: Not recorded in system.md

## Decision
Each subsystem has its own repo.

Reason: team and permission boundaries.

## Consequences
- Stated: team and permission boundaries follow the repo boundaries.
- Inferred: a change that spans subsystems needs one change in each repo, coordinated through the contract (see ADR 0004) and the release tags (section 5.1).
- Inferred: each repo needs its own start command (Operability, NFR-9, NFR-10).

## Affects
S-1, S-2, S-3

## Related
ADR 0002, ADR 0004, ADR 0007, NFR-9, NFR-10
