# ADR 0008: API in Fastify (Node, TypeScript)

## Status
Proposed

## Context
The API (S-2, C-2) is a separate service (see ADR 0002). system.md section 2.2 states the reason: "A different technology from the web frontend, as separate teams would choose. High performance web framework".

## Options considered
- Fastify (Node, TypeScript) (chosen)
- Other options: Not recorded in system.md

## Decision
The API is built in Fastify, on Node, in TypeScript.

Reason: a different technology from the web frontend, as separate teams would choose. It is a high performance web framework.

## Consequences
- Stated: the API uses a different technology from the web frontend (see ADR 0001).
- Inferred: the API has to meet the performance input for 1,000 todos (NFR-7). The framework choice supports that but does not prove it.
- Inferred: the API and the web frontend do not share a framework. They may share the Node and TypeScript runtime.

## Affects
S-2, C-2

## Related
ADR 0001, ADR 0002, NFR-7, NFR-9, NFR-11
