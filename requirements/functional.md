# Functional Requirements: Todo App

Source: `docs/system.md` v0.6. Tags `[r1]`, `[r2]`, `[r3]` give the release that delivers each part.
Section titles refer to `docs/system.md`.

A scenario holds on every frontend that exists at its release or later. Its tag is the first release at which it holds.

## FR-1 Create a todo (CAP-1)

The user can create a todo by submitting one line of text, in the web frontend [r1] and in the iOS frontend [r2]. The new todo then appears in the list [r1].

Rules:
- FR-1.R1 [r1]: A todo text has 1 to 280 characters, counted as Unicode code points, after FR-1.R2 is applied.
- FR-1.R2 [r1]: Leading and trailing Unicode whitespace (characters with the Unicode White_Space property) is removed before the check and before saving. A text that is empty after this is refused.
- FR-1.R3 [r1]: A todo text is one line. A line break inside the text is refused. A line break is LF, CR, U+0085, U+2028 or U+2029.
- FR-1.R4 [r1]: Duplicate texts are allowed.
- FR-1.R5: A refused text creates no todo, and the user sees a message saying it was refused: in the web frontend [r1], in the iOS frontend [r2].
- FR-1.R6 [r1]: A created todo appears in the list with its text, at the position given by FR-2.R2 and FR-2.R3.

Scenarios:
- FR-1.S1 [r1]: Given a frontend is open, when the user submits "Buy milk", then the list shows a todo with the text "Buy milk". (R6)
- FR-1.S2 [r1]: Given a frontend is open, when the user submits a text within the length limit, then the todo is created. (R1)
  Examples:
  - a text of 1 character
  - a text of exactly 280 characters
  - a text of 280 characters, each a single code point outside the Basic Multilingual Plane (for example U+1F600)
- FR-1.S3 [r1]: Given a frontend is open, when the user submits a text over the length limit, then no todo is created and the user sees a message. (R1, R5)
  Examples:
  - a text of 281 characters
  - a text of 281 characters, each a single code point outside the Basic Multilingual Plane
- FR-1.S4 [r1]: Given a frontend is open, when the user submits a text with whitespace at its start or end, then the todo is created with the whitespace removed. (R2, R3)
  Examples:
  - "  Buy milk \t" is saved as "Buy milk"
  - "Buy milk" followed by one line break (any of the five in R3) is saved as "Buy milk"
  - U+00A0, U+2003 or U+3000 on either side of "Buy milk" is saved as "Buy milk"
- FR-1.S5 [r1]: Given a frontend is open, when the user submits a text of 280 characters with whitespace around it, then the todo is created, because the length is checked after the whitespace is removed. (R1, R2)
- FR-1.S6 [r1]: Given a frontend is open, when the user submits a text that is empty after the whitespace is removed, then no todo is created and the user sees a message. (R2, R5)
  Examples:
  - nothing
  - only spaces or tabs
  - only line breaks
  - only U+00A0 or U+3000
- FR-1.S7 [r1]: Given a frontend is open, when the user submits a text with a line break between two words, then no todo is created and the user sees a message. (R3, R5)
  Examples:
  - "Buy" and "milk" separated by LF
  - "Buy" and "milk" separated by CR
  - "Buy" and "milk" separated by U+0085
  - "Buy" and "milk" separated by U+2028
  - "Buy" and "milk" separated by U+2029
- FR-1.S8 [r1]: Given a todo "Buy milk" exists, when the user submits "Buy milk", then a second todo "Buy milk" is created and the list shows both. (R4)

## FR-2 View the list of todos (CAP-2)

The user can view the list of todos and their text, in the web frontend [r1] and in the iOS frontend [r2]. The list is shown when the user opens the frontend [r1].

Rules:
- FR-2.R1 [r1]: The list shows every todo with its text.
- FR-2.R2 [r1]: The list shows the oldest todo first, by created time.
- FR-2.R3 [r1]: If two todos have the same created time, they are ordered by id, ascending, compared as text.
- FR-2.R4 [r1]: If there are no todos, the frontend shows a message instead of the list.
- FR-2.R5 [r1]: A todo stays in the list until it is deleted.

Scenarios:
- FR-2.S1 [r1]: Given three todos exist, when the user opens a frontend, then the list shows the text of all three. (R1)
- FR-2.S2 [r1]: Given three todos with created times T1, T2 and T3, where T1 is the earliest and T3 the latest, when the user opens a frontend, then the list shows them in the order T1, T2, T3. (R2)
- FR-2.S3 [r1]: Given two todos with the same created time and ids X and Y, where X comes before Y when compared as text, when the user opens a frontend, then the list shows X before Y. (R3)
- FR-2.S4 [r1]: Given no todos exist, when the user opens a frontend, then it shows a message and no list. (R4)
- FR-2.S5 [r1]: Given the user created a todo and closed the frontend, when the user opens it again, then the todo is in the list. (R5)

## FR-3 Delete a todo (CAP-3)

The user can delete a todo, in the web frontend [r3] and in the iOS frontend [r3]. The list is then refreshed [r3].

Rules:
- FR-3.R1 [r3]: The user deletes a todo by choosing delete on that todo.
- FR-3.R2 [r3]: After a delete, the frontend refreshes the list as in FR-2, so the deleted todo is no longer shown.
- FR-3.R3 [r3]: If the todo no longer exists, the frontend shows a message, then refreshes the list as in FR-3.R2.

Scenarios:
- FR-3.S1 [r3]: Given todos "A" and "B" exist, when the user chooses delete on "A", then the refreshed list shows only "B". (R1, R2)
- FR-3.S2 [r3]: Given one todo exists, when the user deletes it, then the refreshed view shows the message for an empty list (FR-2.R4). (R2)
- FR-3.S3 [r3]: Given the user sees todo "A" and "A" was already deleted, when the user chooses delete on "A", then the frontend shows a message. (R3)
- FR-3.S4 [r3]: Given the user sees todo "A" and "A" was already deleted, when the user chooses delete on "A", then the frontend shows the refreshed list without "A". (R3)

## FR-4 Use the web frontend (CAP-4)

The user can use the todo capabilities in a web frontend: create and view [r1], delete [r3].

Rules:
- FR-4.R1 [r1]: The web frontend offers create (FR-1) and view (FR-2).
- FR-4.R2 [r3]: The web frontend offers delete (FR-3).

Scenarios: reuses FR-1.S1 and FR-2.S1 for R1, and FR-3.S1 for R2, in the web frontend.

## FR-5 Use the iOS frontend (CAP-5)

The user can use the todo capabilities in an iOS frontend: create and view [r2], delete [r3].

Rules:
- FR-5.R1 [r2]: The iOS frontend offers create (FR-1) and view (FR-2).
- FR-5.R2 [r3]: The iOS frontend offers delete (FR-3).

Scenarios: reuses FR-1.S1 and FR-2.S1 for R1, and FR-3.S1 for R2, in the iOS frontend.

## FR-6 One shared list

There is one list of todos. Every frontend shows the same list [r2].

Rules:
- FR-6.R1 [r2]: A todo created or deleted in one frontend is the same todo in every other frontend. The other frontend shows the change when the user next opens or refreshes its list. Live push between devices is out of scope (see Out-of-Scope).

Scenarios:
- FR-6.S1 [r2]: Given the user created "Buy milk" in one frontend, when the user opens another frontend, then the list shows "Buy milk". (R1)
  Examples:
  - created in the web frontend, opened in the iOS frontend
  - created in the iOS frontend, opened in the web frontend
- FR-6.S2 [r3]: Given the user deleted "Buy milk" in one frontend, when the user next opens another frontend, then the list does not show "Buy milk". (R1)

## FR-7 Todo text is shown as plain text

Todo text is shown as plain text, never as markup, wherever the frontend shows it: in the web frontend [r1] and in the iOS frontend [r2].

Rules:
- FR-7.R1 [r1]: A frontend shows the characters of the text as they were saved, with no formatting, links or markup interpreted.

Scenarios:
- FR-7.S1 [r1]: Given a todo with the text `<b>bold</b> **x** [a](http://example.com)`, when the user views the list in a frontend, then the list shows exactly those characters, with no bold, no link and no tag. (R1)

## FR-8 Failure handling

If the system cannot be reached, or its storage cannot be read or written, the frontend shows a message and changes nothing. This applies to create [r1], view [r1] and delete [r3], in the web frontend [r1] and in the iOS frontend [r2].

Rules:
- FR-8.R1: When an action fails as described above, the frontend shows a message and the list on screen is unchanged: in the web frontend [r1], in the iOS frontend [r2]; for delete [r3].
- FR-8.R2 [r1]: A failed create leaves no half-written todo. This is covered by the non-functional requirement on safety of data, so it has no scenario here.

Scenarios:
- FR-8.S1 [r1]: Given the system cannot be reached, when the user submits a valid text in a frontend, then the frontend shows a message and the list is unchanged. (R1)
- FR-8.S2 [r1]: Given the system cannot be reached, when the user opens a frontend, then it shows a message. (R1)
- FR-8.S3 [r3]: Given the system cannot be reached, when the user chooses delete on a todo, then the frontend shows a message and the todo is still in the list. (R1)

## Trace table

| CAP | FR | Scenarios | Flow | Release |
| --- | --- | --- | --- | --- |
| CAP-1 | FR-1 | FR-1.S1 to FR-1.S8 | F-1 | r1, r2 |
| CAP-2 | FR-2 | FR-2.S1 to FR-2.S5 | F-2 | r1, r2 |
| CAP-3 | FR-3 | FR-3.S1 to FR-3.S4 | F-3 | r3 |
| CAP-4 | FR-4 | reuses FR-1.S1, FR-2.S1, FR-3.S1 | F-1, F-2, F-3 | r1, r3 |
| CAP-5 | FR-5 | reuses FR-1.S1, FR-2.S1, FR-3.S1 | F-1, F-2, F-3 | r2, r3 |
| Capabilities, Rules: "There is one list" | FR-6 | FR-6.S1, FR-6.S2 | F-1, F-2, F-3 | r2, r3 |
| Capabilities, Rules: "plain text" | FR-7 | FR-7.S1 | F-1, F-2 | r1, r2 |
| Error Handling and Fallbacks | FR-8 | FR-8.S1 to FR-8.S3 | F-1, F-2, F-3 | r1, r2, r3 |
