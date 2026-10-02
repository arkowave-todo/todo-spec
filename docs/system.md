# System Description: Todo App

## 0. Document Control
- Version: 0.7.
- Owner: the architect.

## 1. System Boundary

### 1.1. Purpose

A simple todo single-user app. A person can create, view and delete todos.

### 1.2. Capabilities

| ID | Capability | Release Introduced |
| --- | --- | --- |
| CAP-1 | Create a todo from a line of text | r1 |
| CAP-2 | View a list of todos and their text | r1 |
| CAP-3 | Delete a todo and refresh the view | r3 |
| CAP-4 | User can use a web frontend | r1 |
| CAP-5 | User can use an iOS frontend | r2 |

Rules:
- A todo text has 1 to 280 characters, counted as Unicode code points.
- Leading and trailing Unicode whitespace (characters with the Unicode White_Space property) is removed before the check and before saving. A text that is empty after this is refused.
- A todo text is one line. Whitespace is trimmed first (see the previous rule), so only a line break inside the text is refused. A line break is LF, CR, U+0085, U+2028 or U+2029.
- Duplicate texts are allowed.
- The list shows the oldest todo first, by created time. If two todos have the same time, they are ordered by id, ascending, compared as text.
- Todo text is shown as plain text, never as markup.
- There is one list. All frontends show the same list.

### 1.3. Actors

#### 1.3.1. Human Actors

| ID | Actor | Type | What they do |
| --- | --- | --- | --- | 
| A-1 | Todo user | Human | Creates, views and deletes todos through a frontend app | 

#### 1.3.2. External Systems

- None

### 1.4. System Scope

#### 1.4.1. In-Scope

r1
- Create a todo from a line of text (CAP-1)
- View all todos (CAP-2)
- Web app frontend (CAP-4)

r2
- iOS app frontend (CAP-5)

r3
- Delete a todo and refresh view (CAP-3)

#### 1.4.2. Out-of-Scope

- Accounts, login and multiple users. There is a single user app.
- Edit, complete, tag, search or sort todos
- Sync between devices other than through the API
- Offline use, notifications, import and export

## 2. Technical Constraints

### 2.1. Non-functional Requirements

These are inputs to the non-functional requirements. Numbers come in that step.

- **Security:** No login. Single user app designed running on local machine. It is never exposed to the public internet.
- **Safety of data:** a failed create must not leave a half-written todo.
- **Performance:** the list feels instant for up to 1,000 todos.
- **Operability:** each block starts with one command. For the iOS block, that command builds the app and launches it in the Xcode simulator. The API reports if it is healthy.

### 2.2. Technology Stack

Each one becomes an ADR in `architecture/adr/`. All are proposed until the architect accepts them.

| ADR | Decision | Reason |
| --- | --- | --- |
| 0001 | Web frontend in Astro | Simple, and good for learning |
| 0002 | API is a separate service in Next.js | Keeps a real boundary between teams |
| 0003 | One text file per todo, no database | Simple, and enough for this scale |
| 0004 | REST with an OpenAPI contract, owned by `todo-spec` | Teams work in parallel against one contract |
| 0005 | iOS in SwiftUI, tested in the Xcode simulator, no App Store | Keeps r2 small |
| 0006 | One repo per subsystem  | Team and permission boundaries |
| 0007 | One repo for the spec (`todo-spec`) and one repos for the project environment (`todo-platform`) | Team and permission boundaries |

## 3. System Architecture

### 3.1. Subsystems

| ID | Subsystem | Responsibility | Repo |
| --- | --- | --- | --- |
| S-1 | `frontend-web` | user web app frontend to todos | `todo-fe-web` |
| S-2 | `backend` | owns the lifecycle of todos and their storage | `todo-api` |
| S-3 | `frontend-ios` | user iOS app frontend to todos | `todo-fe-ios` |

### 3.2. Interfaces and APIs

| ID | Interface | Responsibility | Implemented By | Clients To Interface |
| --- | --- | --- | --- | --- | 
| I-1 | `api-todo` | lifecycle management of todo text (create, view, delete) | S-2 | S-1, S-3 |

Rules:
- The API contract lives in `todo-spec`. It is the only link between frontends and the API.
- REST API, described in OpenAPI
- Both frontends use the same API. 

### 3.3. AI and Data Architecture

| ID | Data | Fields | Owner | Storage |
| --- | --- | --- | --- | --- |
| D-1 | Todo | id (made by the system), text, created time | The API | One text file per todo |

Rules:
- Only the API reads and writes the files. Frontends never touch storage.
- The file name comes from the generated id, never from the todo text.
- Ids are unique. Their format is set in the API contract.
- A todo stays until it is deleted. There is no backup.

## 4. Component Design

### 4.1. Software Components

| ID | Subsystem Owner | Component Name | Technology Pattern | Technology Type |
| --- | --- | --- | --- | --- | 
| C-1 | S-1 | `web-fe` | web application | Astro |
| C-2 | S-2 | `todo-api` | api server | Next.js |
| C-3 | S-2 | `todo-store` | files | Text file |
| C-4 | S-3 | `ios-fe` | iOS application | SwiftUI |

### 4.2. Data Flows

#### F-1 Create a todo
1. The user types a text and submits it in the frontend.
2. The frontend sends the text to the API.
3. The API checks the text, makes an id and writes the file.
4. The API returns the new todo.
5. The frontend shows the todo in the list.

Error: 
- if the text is refused (empty, too long, or it has a line break), the API refuses it and the frontend shows a message.

#### F-2 View todos
1. The user opens the frontend.
2. The frontend asks the API for the list.
3. The API reads the files and returns the list.
4. The frontend shows the list.

Exception:
- if there are no todos then show a message.

#### F-3 Delete a todo
1. The user chooses delete on a todo.
2. The frontend asks the API to delete it by id.
3. The API removes the file and confirms. If the id is unknown, it says so.
4. The frontend runs F-2 from step 2 to refresh the list.

Error:
- if the id is unknown, the frontend shows a message, then refreshes the list as in step 4.

### 4.3. Error Handling and Fallbacks

- If the API cannot be reached, or a file cannot be read or written, the frontend shows a message and changes nothing.
- The API never leaves a partial todo.

## 5. Release Strategy
### 5.1. Release Plan

| Release | Actors | Components New | Components Updated | Flows Delivered |
| --- | --- | --- | --- | --- |
| r1 | A-1 | C-1, C-2, C-3 | none | F-1, F-2 |
| r2 | A-1 | C-4 | none | F-1, F-2 |
| r3 | A-1 | none | C-1, C-2, C-4 | F-3 |

Each release gets a tag in `todo-spec`. Code repos pin to that tag.

### 5.2. Test and Validation Plan

- N/A

## 6. Glossary and Appendix

- **Todo**: one line of text that a person wants to remember.
- **Block**: a part of the system that one team owns. Each block has one repo.
- **Contract:** the written description of the API. Frontends and the API both follow it.
- **Release:** a named step (r1, r2, r3) with a git tag in `todo-spec`.


