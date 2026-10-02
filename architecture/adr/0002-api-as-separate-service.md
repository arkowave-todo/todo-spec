# ADR 0002: API is a separate service, not part of the web frontend

## Status
Proposed

## Context
The system has two frontends (S-1, S-3) that show the same list (system.md section 1.2). They use the same API (section 3.2, I-1). system.md section 2.2 states the reason: "Keeps a real boundary between teams".

## Options considered
- API as a separate service (chosen)
- API as part of the web frontend: named in system.md only as the rejected form ("not part of the web frontend")
- Other options: Not recorded in system.md

## Decision
The API (S-2) is a separate service. It is not part of the web frontend (S-1).

Reason: it keeps a real boundary between teams.

## Consequences
- Stated: a real boundary exists between the teams that own the API and the frontends.
- Inferred: the iOS frontend can use the API without going through the web frontend.
- Inferred: the API has to be started on its own (see NFR-9) and the frontends must handle an unreachable API (Error Handling and Fallbacks).
- Inferred: a frontend cannot call into API code. It needs a contract (see ADR 0004).

## Affects
S-1, S-2, S-3, C-1, C-2, C-4, I-1

## Related
ADR 0004, ADR 0006, ADR 0008, NFR-9, NFR-11, FR-6, FR-8
