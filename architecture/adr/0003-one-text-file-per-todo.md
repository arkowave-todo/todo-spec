# ADR 0003: One text file per todo, no database

## Status
Proposed

## Context
The API owns the storage of todos (S-2, D-1). system.md section 2.2 states the reason: "Simple, and enough for this scale". The scale input in section 2.1 is a list that feels instant for up to 1,000 todos.

## Options considered
- One text file per todo (chosen)
- Database: named in system.md only as the rejected form ("no database")
- Other options: Not recorded in system.md

## Decision
Each todo is stored as one text file. There is no database.

Reason: simple, and enough for this scale.

## Consequences
- Stated: storage is simple.
- Stated: only the API reads and writes the files. The file name comes from the generated id (D-1 rules).
- Stated: a todo stays until it is deleted. There is no backup (D-1 rules).
- Inferred: the API has to make a create atomic itself, so that a failed create leaves no half-written todo (Safety of data; NFR-3, NFR-4).
- Inferred: viewing the list means reading every file. This has to meet the performance input for 1,000 todos (NFR-7).
- Inferred: no query, transaction or backup features come with the storage.

## Affects
S-2, C-2, C-3

## Related
D-1, NFR-2, NFR-3, NFR-4, NFR-5, NFR-6, NFR-7, FR-2, FR-8
