# Non-functional Requirements: Todo App

Source: `docs/system.md` v0.6, section "Non-functional Requirements" and the data rules in "AI and Data Architecture". Tags `[r1]`, `[r2]`, `[r3]` give the release from which each NFR must hold.

`docs/system.md` gives only two numbers: the scale of 1,000 todos and one start command per block. A value that comes from `system.md` is marked Given. Every other number is marked Proposed and waits for the architect's decision.

Stages: inner loop (developer machine, per change), outer loop (CI, per merge), non-functional test (dedicated run against a built system), release (checked before a tag).

Functional rules are not repeated here. They are referred to by ID.

## Reference environment (Proposed)

Every condition below refers to one of these profiles. A result counts only when it is measured on the profile named in the condition.

| Profile | Used for | CPU cores | Memory | Storage | OS |
| --- | --- | --- | --- | --- | --- |
| RE-1 | S-1 and S-2 (web, API) | 4 | 8 GB | local SSD, 20 GB free | Linux, current LTS release |
| RE-2 | S-3 (iOS in the Xcode simulator), with the API on the same machine | 8 (Apple silicon) | 16 GB | local SSD, 50 GB free | macOS, current release |

A p95 needs at least 100 samples. Where 100 runs cost too much, the NFR states a maximum instead of a percentile.

## Security

### NFR-1 Local access only [r1]
- Applies to: S-2 (`backend`).
- Metric: number of non-loopback network addresses on which the API accepts a connection.
- Target: 0 in the default configuration. Proposed.
- Condition: RE-1. API started with its one start command (NFR-9), no extra configuration, on a machine that has a non-loopback address.
- Measured by: connect to the API port through the machine's non-loopback address and expect a refused connection; list the listening sockets of the API process and expect loopback only.
- Stage: non-functional test.
- Basis: Security: "never exposed to the public internet". "No login" is a design decision, so it has no metric.

### NFR-2 Storage path isolation [r1]
- Applies to: S-2.
- Metric: number of files created, read, changed or removed outside the store folder.
- Target: 0. Proposed.
- Condition: RE-1. Todo texts, and ids sent in a delete request [r3], that hold path syntax: `../`, `/`, `\`, `.`, `..`, a NUL character, a name of 255 or more characters, and a text of 280 characters.
- Measured by: run the create and delete operations with these inputs against a store inside an otherwise empty parent folder; compare the file tree of the parent before and after. Check that every file name in the store matches the id format of the API contract and none is derived from the text.
- Stage: outer loop.
- Basis: D-1 rules "The file name comes from the generated id, never from the todo text" and "Only the API reads and writes the files".
- Related: valid text with path syntax is still a valid todo (FR-1.R1 to FR-1.R3).

## Safety of data

### NFR-3 No partial todo after a failed write [r1]
- Applies to: S-2.
- Metric: number of partial todos, and number of changes to the list, after a failed create.
- Target: 0 of each, at every injected failure point. Proposed set of points: before the write starts, during the write, after the write and before the response.
- Condition: RE-1. Store holds 10 todos; a write failure is injected at each point; the failing create is followed by a list request.
- Measured by: fault-injection test. The list has the same 10 todos, and the store folder holds no file that is not a complete todo.
- Stage: outer loop.
- Covers: FR-8.R2.

### NFR-4 No partial todo after a crash during create [r1]
- Applies to: S-2.
- Metric: number of partial todos after the API process is killed during a create.
- Target: 0 in 200 runs. Proposed.
- Condition: RE-1. The process is killed with SIGKILL at a random moment while creates of 280-character texts run in a loop; the API is then restarted.
- Measured by: after each restart, request the list; every todo listed is complete and has the text that was sent; the API starts without manual repair.
- Stage: non-functional test.
- Covers: FR-8.R2 and the section "Error Handling and Fallbacks" ("The API never leaves a partial todo").

### NFR-5 A created todo survives a restart [r1]
- Applies to: S-2.
- Metric: number of todos lost or changed after the API is stopped and started again.
- Target: 0. Proposed.
- Condition: RE-1. 1,000 todos; both a normal stop and SIGKILL; killed immediately after the create response is received.
- Measured by: list before and after the restart; the two lists are equal. For the kill case, every todo whose create response was received is in the list.
- Stage: non-functional test.
- Basis: D-1 rule "A todo stays until it is deleted". The user-visible rule is FR-2.R5. There is no backup, so nothing else protects the data.

### NFR-6 A failed delete leaves the todo intact [r3]
- Applies to: S-2.
- Metric: number of todos damaged or changed by a failed delete.
- Target: 0. Proposed.
- Condition: RE-1. A failure is injected before, during and after the removal of the file; store holds 10 todos.
- Measured by: fault-injection test. After a failure before or during the removal, the todo is listed with its original text, or it is absent; it is never listed with other text or without text. The other 9 todos are unchanged.
- Stage: outer loop.
- Basis: "Error Handling and Fallbacks". The user-visible behaviour is FR-8.R1 and FR-8.S3.

## Performance

### NFR-7 List response time of the API [r1]
- Applies to: S-2.
- Metric: server response time of the list request, 95th percentile.
- Target: at most 100 ms. Proposed. Scale of 1,000 todos is Given. The first request after a start: at most 500 ms. Proposed.
- Condition: RE-1. 1,000 todos, each with a 280-character text; API and test client on the same machine; 100 requests after 10 warm-up requests; also the first request after a start.
- Measured by: timing script that records each response time. The first request after a start is reported separately from the 100 requests.
- Stage: non-functional test.
- Basis: Performance: "the list feels instant for up to 1,000 todos".

### NFR-8 Time to a visible list in the frontends [r1 web, r2 iOS]
- Applies to: S-1 [r1], S-3 [r2], with S-2.
- Metric: time from opening the frontend to the full list on screen. Web: 95th percentile. iOS: maximum.
- Target: web at most 500 ms. iOS at most 1,000 ms. Proposed.
- Condition: RE-1 for web, RE-2 for iOS. 1,000 todos with 280-character texts; API on the same machine; web frontend in a current desktop browser; iOS frontend in the Xcode simulator. Web: 100 runs. iOS: 20 runs, because a simulator run costs too much to repeat 100 times.
- Measured by: browser automation (web) and a UI test (iOS) that record the time until the 1,000th todo is displayed.
- Stage: non-functional test.
- Basis: Performance: "the list feels instant for up to 1,000 todos". NFR-7 is one part of this budget.

## Operability

### NFR-9 One command starts a block, web and API [r1]
- Applies to: S-1, S-2.
- Metric: number of commands from a clean checkout to a running block, and time until it is ready.
- Target: 1 command (Given: "each block starts with one command"); ready in at most 30 s. Proposed.
- Condition: RE-1. Clean checkout of the repo with the documented tools installed; no other manual step, and no edit of any file.
- Measured by: scripted run in a clean environment that runs the command and waits for ready. For S-2, ready is a healthy answer from NFR-11. For S-1, ready is the page answering a request.
- Stage: release.

### NFR-10 One command starts the iOS block [r2]
- Applies to: S-3.
- Metric: number of commands from a clean checkout to the app running in the Xcode simulator.
- Target: 1 command (Given). Time to ready is at most 120 s. Proposed.
- Condition: RE-2. Clean checkout, Xcode and a simulator installed, API running (NFR-9).
- Measured by: scripted run that runs the command and waits until the app shows the list or the empty-list message.
- Stage: release.
- Note: ADR 0005 says iOS is tested in the simulator with no App Store, so the command covers build and launch in the simulator.

### NFR-11 The API reports its health [r1]
- Applies to: S-2.
- Metric: share of storage states for which the health answer is correct, and health response time, 95th percentile.
- Target: 100% correct in the states below; at most 100 ms. Proposed.
- Condition: RE-1. States tested: store readable and writable (healthy), store folder missing, store folder not readable, store folder not writable (each unhealthy). 100 requests per state.
- Measured by: automated test that sets each state and reads the health answer. The shape of the answer is set in the API contract (I-1).
- Stage: outer loop.
- Basis: Operability: "The API reports if it is healthy".

## Trace table

| Input | NFR | Applies to | Stage | Release |
| --- | --- | --- | --- | --- |
| Security: never exposed to the public internet | NFR-1 | S-2 | non-functional test | r1 |
| D-1 rules: file name from id, only the API touches files | NFR-2 | S-2 | outer loop | r1 |
| Safety of data: failed create leaves no half-written todo; FR-8.R2 | NFR-3 | S-2 | outer loop | r1 |
| Safety of data; Error Handling and Fallbacks: no partial todo; FR-8.R2 | NFR-4 | S-2 | non-functional test | r1 |
| D-1 rule: a todo stays until deleted; FR-2.R5 | NFR-5 | S-2 | non-functional test | r1 |
| Error Handling and Fallbacks; FR-8.R1, FR-8.S3 | NFR-6 | S-2 | outer loop | r3 |
| Performance: list instant for 1,000 todos | NFR-7 | S-2 | non-functional test | r1 |
| Performance: list instant for 1,000 todos | NFR-8 | S-1, S-3 (with S-2) | non-functional test | r1 (S-1), r2 (S-3) |
| Operability: each block starts with one command | NFR-9 | S-1, S-2 | release | r1 |
| Operability: each block starts with one command | NFR-10 | S-3 | release | r2 |
| Operability: the API reports if it is healthy | NFR-11 | S-2 | outer loop | r1 |
