# ADR 0007: One repo for the spec (`todo-spec`) and one repo for the project environment (`todo-platform`)

## Status
Proposed

## Context
Besides the subsystem repos (see ADR 0006), the project has a spec and a project environment. The spec repo owns the API contract. The project environment (`todo-platform`) holds the sandbox image, policy files, provider profiles and scripts (section 6, Glossary). system.md section 2.2 states the reason: "Team and permission boundaries".

## Options considered
- One repo for the spec and one repo for the project environment (chosen)
- Other options: Not recorded in system.md

## Decision
The spec lives in `todo-spec`. The project environment lives in `todo-platform`. They are two repos.

Reason: team and permission boundaries.

## Consequences
- Stated: team and permission boundaries separate the spec from the project environment.
- Stated: each release gets a tag in `todo-spec`, and code repos pin to that tag (section 5.1).
- Inferred: changes to the sandbox, policy or scripts do not need a change to the spec, and the reverse.

## Affects
`todo-spec`, `todo-platform`

## Related
ADR 0004, ADR 0006
