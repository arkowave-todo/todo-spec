# ADR 0001: Web frontend in Astro

## Status
Proposed

## Context
The system needs a web frontend (CAP-4, S-1, C-1) from r1. system.md section 2.2 states the reason for the choice: "Simple, and good for learning".

## Options considered
- Astro (chosen)
- Other options: Not recorded in system.md

## Decision
The web frontend is built in Astro.

Reason: simple, and good for learning.

## Consequences
- Stated: the web frontend is simple to build.
- Stated: the team learns Astro.
- Inferred: the web frontend uses a different technology from the API (see ADR 0008) and from the iOS frontend (see ADR 0005).

## Affects
S-1, C-1

## Related
CAP-4, FR-4, NFR-8, NFR-9, ADR 0002, ADR 0008
