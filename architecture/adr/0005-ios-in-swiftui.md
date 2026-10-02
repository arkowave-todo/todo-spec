# ADR 0005: iOS in SwiftUI, tested in the Xcode simulator, no App Store

## Status
Proposed

## Context
The system has an iOS frontend (CAP-5, S-3, C-4) from r2. system.md section 2.2 states the reason: "Keeps r2 small".

## Options considered
- SwiftUI, tested in the Xcode simulator, no App Store (chosen)
- Other options: Not recorded in system.md

## Decision
The iOS frontend is built in SwiftUI. It is tested in the Xcode simulator. It is not published to the App Store.

Reason: keeps r2 small.

## Consequences
- Stated: r2 stays small.
- Stated: for the iOS block, one command builds the app and launches it in the Xcode simulator (Operability).
- Inferred: building and testing the iOS block needs a machine with Xcode.
- Inferred: the app cannot be installed on a user's device through the App Store. It runs in the simulator only.

## Affects
S-3, C-4

## Related
CAP-5, FR-5, NFR-8, NFR-10, ADR 0004
