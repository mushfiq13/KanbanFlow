# KanbanFlow — User Story Backlog

PM-mode artifact. Each story is sized for solo implementation in 1–3 days.
Stories are ordered by dependency; you can implement them in any order within an
Epic, but Epics should be tackled top-to-bottom.

**Legend**:

- **SP** = Story Points (Fibonacci: 1, 2, 3, 5, 8)
- **Priority**: MoSCoW (Must / Should / Could / Won't for this MVP)
- **AC** = Acceptance Criteria in Given/When/Then form
- **Deps** = Stories that must complete first

---

## EPIC-1: Project Bootstrap

### US-1.1 Install toolchain

**As a** developer **I want** PHP 8.3+, Composer, Node 20+, and PostgreSQL 16+
installed locally **so that** I can scaffold the backend and frontend. **SP**: 1
**Priority**: Must **Deps**: none **AC**:

- Given a fresh macOS/Linux machine
- When I check the runtime versions
- Then each tool prints a version at or above its required minimum

---

### US-1.2 Scaffold backend project

**As a** developer **I want** a Laravel 12 backend project in an `api/` folder
**so that** I have a runnable HTTP API skeleton. **SP**: 2 **Priority**: Must
**Deps**: US-1.1 **AC**:

- Given Composer is installed
- When I create the project in the project root
- Then an `api/` folder contains a Laravel 12 project
- And the framework version command reports Laravel 12.x
- And the API installer command is run and produces a routes file for the API

---

### US-1.3 Configure database connection

**As a** developer **I want** the backend connected to a local PostgreSQL
database **so that** migrations can run. **SP**: 2 **Priority**: Must **Deps**:
US-1.2 **AC**:

- Given PostgreSQL is running locally
- When I create a `kanbanflow` database and a user
- And I configure the database credentials in the backend environment file
- Then running the migration command succeeds
- And the database inspection command lists the migrated tables

---

### US-1.4 Scaffold frontend project

**As a** developer **I want** a Next.js 15 frontend project in a `web/` folder
**so that** I have a runnable React/TypeScript skeleton. **SP**: 2 **Priority**:
Must **Deps**: US-1.1 **AC**:

- Given Node 20+ is installed
- When I create the project with TypeScript, Tailwind, App Router, ESLint, and a
  `src/` directory
- Then a `web/` folder contains a Next.js 15 App Router project
- And the dev server starts on port 3000
- And `http://localhost:3000` shows the default Next.js page

---

### US-1.5 Add Docker Compose for local stack

**As a** developer **I want** one `docker compose up` to start Postgres + Redis
**so that** onboarding is fast. **SP**: 3 **Priority**: Should **Deps**: US-1.3
**AC**:

- Given Docker Desktop is installed
- When I run `docker compose up -d` from the project root
- Then Postgres is reachable on `localhost:5432`
- And Redis is reachable on `localhost:6379`
- And the backend migration command succeeds against the Postgres container

**Notes**: Only Postgres + Redis in the container. Run the backend and frontend
on the host for fast dev iteration.

---

### EPIC-1 exit criteria

- [ ] `api/` and `web/` folders exist side-by-side
- [ ] Backend starts via `cd api && php artisan serve` (port 8000)
- [ ] Frontend starts via `cd web && pnpm dev` (port 3000)
- [ ] Database connection works
- [ ] `git init` done, `.gitignore` covers `vendor/`, `node_modules/`, `.env`

---

## EPIC-2: Auth & User Management

### US-2.1 User registration

**As a** new user **I want** to register with email + password **so that** I can
start using KanbanFlow. **SP**: 3 **Priority**: Must **Deps**: US-1.2, US-1.3
**AC**:

- Given a registration request with name, email, password, and password
  confirmation
- When the input is valid
- Then the response indicates success and returns the created user plus an
  authentication token
- And the password is stored as a secure hash (never plain text)
- And duplicate email is rejected with a clear error
- And the token is the kind of token the rest of the API expects on subsequent
  authenticated requests

---

### US-2.2 User login

**As a** registered user **I want** to log in with email + password **so that**
I receive an authentication token. **SP**: 2 **Priority**: Must **Deps**: US-2.1
**AC**:

- Given a login request with email and password
- When credentials are correct
- Then the response returns the user plus a fresh authentication token
- When credentials are wrong
- Then the response returns a clear "invalid credentials" error
- And login is rate-limited to 5 attempts per minute per IP

---

### US-2.3 User logout

**As a** logged-in user **I want** to revoke my current session token **so
that** I'm logged out. **SP**: 1 **Priority**: Must **Deps**: US-2.1 **AC**:

- Given an authenticated request
- When the user triggers logout
- Then the response indicates success
- And the current authentication token can no longer be used

---

### US-2.4 Current user endpoint

**As a** frontend **I want** an endpoint that returns the current user **so
that** I can verify a stored token is still valid. **SP**: 1 **Priority**: Must
**Deps**: US-2.1 **AC**:

- Given an authenticated request
- When the endpoint is called
- Then the response returns the current user's id, name, email, and avatar URL
- When unauthenticated, an unauthorized error is returned

---

### US-2.5 Frontend login page

**As a** user **I want** a login form **so that** I can sign in. **SP**: 3
**Priority**: Must **Deps**: US-2.2, US-1.4 **AC**:

- Given the `/login` page
- When I submit valid credentials
- Then the token is stored in localStorage under `kanbanflow_token`
- And I'm redirected to `/boards`
- When credentials are wrong, an inline error appears
- The form uses a form library with schema-based validation

---

### US-2.6 Frontend register page

**As a** new user **I want** a registration form **so that** I can create an
account. **SP**: 3 **Priority**: Must **Deps**: US-2.1, US-1.4 **AC**:

- Given the `/register` page
- When I submit valid input
- Then I'm registered and logged in automatically
- When email is taken, an inline error appears
- Password field shows a strength indicator (optional polish)

---

### US-2.7 API client with auth interceptor

**As a** developer **I want** a centralized API client **so that** every API
call includes the auth token automatically. **SP**: 2 **Priority**: Must
**Deps**: US-1.4 **AC**:

- Given a request from any page
- When the API client makes a call
- Then the auth token is attached if one exists in localStorage
- When a 401 response comes back, the token is cleared and the user is
  redirected to `/login`

---

### EPIC-2 exit criteria

- [ ] Can register, log in, log out via API
- [ ] Can register, log in, log out via UI
- [ ] Stored token works across browser tabs
- [ ] Unauthenticated API calls return an unauthorized error
- [ ] Frontend redirects unauthenticated users to `/login`

---

## EPIC-3: Boards

### US-3.1 Create board

**As a** user **I want** to create a board **so that** I can start organizing
work. **SP**: 2 **Priority**: Must **Deps**: US-2.1 **AC**:

- Given an authenticated user
- When the user creates a board with a title
- Then the response indicates success and returns the new board
- And the creator is recorded as the board owner in the membership table

---

### US-3.2 List user's boards

**As a** user **I want** to see all boards I own or am a member of **so that** I
can pick one to open. **SP**: 2 **Priority**: Must **Deps**: US-3.1 **AC**:

- Given an authenticated user
- When the boards list is requested
- Then the response returns all boards the user owns OR is a member of
- And each board shows id, title, background, the user's role on it, and
  last-updated timestamp
- Boards are sorted by last-updated, most recent first

---

### US-3.3 Get board detail

**As a** user **I want** to open a single board **so that** I see all its lists
and cards. **SP**: 3 **Priority**: Must **Deps**: US-4.1, US-5.1 **AC**:

- Given the user is a member of the board
- When the board detail is requested
- Then the response returns the full board tree: the board itself, the owner,
  the members, the labels, and all lists with their cards in order
- The response is efficiently constructed (no N+1 query pattern)

---

### US-3.4 Rename board

**As a** board owner/admin **I want** to rename a board **so that** I can keep
its title current. **SP**: 1 **Priority**: Must **Deps**: US-3.1 **AC**:

- Given the user is the owner or admin of the board
- When the board is updated with a new title (and optionally description and
  background)
- Then the response returns the updated board
- And the new title is between 1 and 120 characters
- When the user is not owner/admin, an unauthorized error is returned

---

### US-3.5 Archive board (soft delete)

**As a** board owner **I want** to archive a board **so that** I can clean up my
workspace without losing history. **SP**: 1 **Priority**: Must **Deps**: US-3.1
**AC**:

- Given the user is the owner of the board
- When the user archives the board
- Then the response indicates success and the board is marked archived (hidden
  from the default list but recoverable)
- And a separate restore action brings the board back to active state

---

### US-3.6 Frontend boards list page

**As a** user **I want** a `/boards` page showing all my boards **so that** I
can pick one. **SP**: 3 **Priority**: Must **Deps**: US-3.2, US-2.7 **AC**:

- The page renders a card or link per board
- Clicking a board navigates to its detail page
- Empty state: a friendly "No boards yet. Create one." message
- Includes a "Create board" button
- Data fetched via a query library and cached

---

### EPIC-3 exit criteria

- [ ] Can create/rename/archive a board via UI
- [ ] Can see all boards I belong to
- [ ] Archived boards don't appear in the default list

---

## EPIC-4: Lists

### US-4.1 Create list in board

**As a** board member **I want** to create a list (column) in a board **so
that** I can group cards by stage. **SP**: 2 **Priority**: Must **Deps**: US-3.1
**AC**:

- Given the user is a member of the board
- When the user creates a list with a title
- Then the response returns the new list
- And the new list is appended to the end of the board's existing lists
- When the user is not a member, an unauthorized error is returned

---

### US-4.2 Rename list

**As a** board member **I want** to rename a list **so that** I can keep its
title current. **SP**: 1 **Priority**: Must **Deps**: US-4.1 **AC**:

- Given the user is a member of the board
- When the user updates the list's title
- Then the response returns the updated list
- When the user is not a member, an unauthorized error is returned

---

### US-4.3 Delete list (soft)

**As a** board owner/admin **I want** to delete a list **so that** I can remove
an unused column. **SP**: 1 **Priority**: Must **Deps**: US-4.1 **AC**:

- Given the user is owner/admin of the board
- When the user deletes the list
- Then the list is marked deleted (hidden from view but recoverable)
- And all cards in the list are also marked deleted
- When the user is not owner/admin, an unauthorized error is returned

---

### US-4.4 Reorder lists via drag

**As a** board member **I want** to drag a list to a new position **so that** I
can reorder my workflow stages. **SP**: 3 **Priority**: Must **Deps**: US-4.1
**AC**:

- Given the user is a member of the board
- When the user moves a list to a new position
- Then the response returns success
- And all other lists' positions are updated to maintain a contiguous, gap-free
  ordering
- Other connected clients see the move in real time

---

### US-4.5 WIP limit per list (visual)

**As a** board member **I want** to set a WIP (work-in-progress) limit on a list
**so that** I can see when the list is overloaded. **SP**: 2 **Priority**: Must
**Deps**: US-4.2 **AC**:

- Given the user is a member of the board
- When the user sets an optional integer limit on a list
- Then the column header shows the current count vs. the limit (e.g. "3 / 5")
- When the count exceeds the limit, the counter turns red as a visual warning
- The system does not block cards from being added; the limit is advisory only

---

### EPIC-4 exit criteria

- [ ] Can create/rename/delete/reorder lists
- [ ] WIP limit displays and turns red on overflow

---

## EPIC-5: Cards

### US-5.1 Create card in list

**As a** board member **I want** to create a card in a list **so that** I can
capture work to be done. **SP**: 2 **Priority**: Must **Deps**: US-4.1 **AC**:

- Given the user is a member of the board
- When the user creates a card with a title (and optional description, due date,
  cover color, and priority)
- Then the response returns the new card
- And the new card is appended to the end of the list
- When the user is not a member, an unauthorized error is returned

---

### US-5.2 Edit card

**As a** board member **I want** to edit a card's fields **so that** I can keep
its details current. **SP**: 2 **Priority**: Must **Deps**: US-5.1 **AC**:

- Given the user is a member of the board
- When the user updates any of the card's fields (title, description, due date,
  cover color, priority)
- Then the response returns the updated card
- And the priority, if provided, must be one of `low`, `medium`, `high`,
  `urgent`, or null
- And the cover color, if provided, must be a valid color value
- Other connected clients see the change in real time

---

### US-5.3 Delete card (soft)

**As a** board member **I want** to delete a card **so that** I can remove it
without losing history. **SP**: 1 **Priority**: Must **Deps**: US-5.1 **AC**:

- Given the user is a member of the board
- When the user deletes the card
- Then the response indicates success and the card is marked deleted (hidden but
  recoverable)
- Other connected clients see the deletion in real time

---

### US-5.4 Move card within or across lists

**As a** board member **I want** to drag a card to a new position **so that** I
can move work through the workflow. **SP**: 3 **Priority**: Must **Deps**:
US-5.1 **AC**:

- Given the user is a member of both the source and target boards
- When the user moves a card to a target list and target position
- Then the response returns success
- And all positions in the source list and target list are updated to maintain a
  contiguous, gap-free ordering
- And positions are renumbered automatically when gaps fall below 1
- Other connected clients see the move in real time
- When the user is not a member of the target board, an unauthorized error is
  returned

---

### US-5.5 Frontend kanban board page

**As a** user **I want** to see a board with all its lists and cards **so that**
I can drag cards around. **SP**: 5 **Priority**: Must **Deps**: US-3.3, US-5.1
**AC**:

- The board page renders horizontally scrolling list columns
- Each column shows its cards stacked vertically
- Each card displays its title, priority dot, due date (if set), label chips,
  assignee avatars, and comment count
- Clicking a card opens a card detail modal
- Each list has an "Add card" button at the bottom
- The board has an "Add list" button at the far right

---

### US-5.6 Drag-and-drop interaction

**As a** user **I want** to drag cards and lists with my mouse or finger **so
that** I can rearrange them quickly. **SP**: 5 **Priority**: Must **Deps**:
US-5.5 **AC**:

- Cards drag within a list and across lists
- Lists drag within the board
- On drop, the UI updates immediately (optimistic) and a backend mutation is
  sent
- If the mutation fails, the UI rolls back to its previous state
- The drag library is a modern, accessible library

---

### US-5.7 Card detail modal

**As a** user **I want** to open a card to see and edit all its details **so
that** I have full information in one place. **SP**: 5 **Priority**: Must
**Deps**: US-5.2 **AC**:

- Clicking a card opens a modal
- The modal shows: title (editable inline), description (markdown, editable),
  due date, cover color, priority, labels, assignees, watchers, comments, and
  activity log
- The modal closes on Escape, click outside, or the close button
- Edits save automatically after the user stops typing (debounced)

---

### EPIC-5 exit criteria

- [ ] Can create/edit/delete cards
- [ ] Cards drag within and across lists
- [ ] Lists drag within board
- [ ] Positions persist after refresh
- [ ] Card modal shows all editable fields

---

## EPIC-6: Labels

### US-6.1 Create label in board

**As a** board member **I want** to create a label (color tag) in a board **so
that** I can categorize cards. **SP**: 2 **Priority**: Must **Deps**: US-3.1
**AC**:

- Given the user is a member of the board
- When the user creates a label with a name and color
- Then the response returns the new label
- And the color must be a valid color value

---

### US-6.2 List / edit / delete label

**As a** board member **I want** to view, rename, recolor, and delete labels
**so that** I can manage my categorization scheme. **SP**: 2 **Priority**: Must
**Deps**: US-6.1 **AC**:

- Listing labels returns all labels defined for the board
- Editing updates the label's name and/or color
- Deleting removes the label and detaches it from every card that had it

---

### US-6.3 Attach/detach label to card

**As a** board member **I want** to attach or detach labels on a card **so
that** I can tag the card appropriately. **SP**: 2 **Priority**: Must **Deps**:
US-6.1, US-5.1 **AC**:

- Attaching a label to a card that already has it is a no-op success
- Detaching a label that isn't on the card is a no-op success
- Attaching/detaching returns success

---

### US-6.4 Label chips on card UI

**As a** user **I want** to see labels as colored chips on cards **so that** I
can scan categorization at a glance. **SP**: 2 **Priority**: Must **Deps**:
US-6.3 **AC**:

- The card front shows label chips (small colored pills)
- The card modal includes a full label picker to add or remove labels
- Labels also work as a quick-filter criterion (see US-12.x)

---

### EPIC-6 exit criteria

- [ ] Can create/rename/delete labels per board
- [ ] Can attach/detach labels on cards
- [ ] Labels show as colored chips on cards

---

## EPIC-7: Assignees

### US-7.1 Assign user to card

**As a** board member **I want** to assign a user to a card **so that** it's
clear who owns the work. **SP**: 2 **Priority**: Must **Deps**: US-3.1, US-5.1
**AC**:

- Given a target user who is a member of the board
- When the user is assigned to the card
- Then the response indicates success
- Assigning someone already on the card is a no-op success
- Trying to assign a non-member is rejected with a clear error

---

### US-7.2 Unassign user from card

**As a** board member **I want** to unassign a user from a card **so that** I
can remove an incorrect assignee. **SP**: 1 **Priority**: Must **Deps**: US-7.1
**AC**:

- When the user is unassigned from the card
- Then the response indicates success

---

### US-7.3 Assignee avatars on card UI

**As a** user **I want** to see who is assigned to a card at a glance **so
that** I know ownership without opening the card. **SP**: 2 **Priority**: Must
**Deps**: US-7.1 **AC**:

- The card front shows up to 3 assignee avatars (overlapping)
- When there are more than 3 assignees, a `+N` indicator is shown
- In the card modal, clicking an avatar reveals the option to remove that
  assignee

---

### EPIC-7 exit criteria

- [ ] Can assign/unassign board members to cards
- [ ] Assignee avatars show on card front and in modal

---

## EPIC-8: Subtasks

### US-8.1 Subtask data model

**As a** developer **I want** subtasks to be modeled as cards with a parent
reference **so that** I can reuse the card schema. **SP**: 1 **Priority**: Must
**Deps**: US-5.1 **AC**:

- The cards table gains a nullable parent reference
- A subtask is a card whose parent reference points to another card
- A subtask of a subtask is not allowed (parent must itself have no parent)

---

### US-8.2 Create subtask

**As a** board member **I want** to add a subtask to a card **so that** I can
break work down. **SP**: 2 **Priority**: Must **Deps**: US-8.1 **AC**:

- Given a parent card
- When the user creates a subtask with a title (and optional fields)
- Then the response returns the new subtask linked to the parent
- The parent card cannot itself be a subtask (no nesting beyond one level)

---

### US-8.3 List subtasks for card

**As a** board member **I want** to see the subtasks for a card **so that** I
can track progress. **SP**: 1 **Priority**: Must **Deps**: US-8.2 **AC**:

- The response returns the parent card's subtasks, ordered by position
- Each subtask shows its id, title, and completion state

---

### US-8.4 Subtask completion

**As a** board member **I want** to mark a subtask as complete or incomplete
**so that** progress is tracked. **SP**: 2 **Priority**: Must **Deps**: US-8.2
**AC**:

- The subtask has a completion state (complete / incomplete)
- Setting the state to complete records the completion time
- Setting it back to incomplete clears the completion time
- The parent card shows a summary like "3 / 5 done"

---

### US-8.5 Subtask UI in card modal

**As a** user **I want** a subtasks section in the card detail modal **so that**
I can view and manage subtasks inline. **SP**: 3 **Priority**: Must **Deps**:
US-8.2 **AC**:

- The card modal includes a "Subtasks" section
- The section lists each subtask with a checkbox and title (clicking the title
  opens that subtask's own detail modal)
- An inline input lets the user add a new subtask
- A progress indicator shows how many subtasks are done out of the total

---

### EPIC-8 exit criteria

- [ ] Can add subtasks under any card
- [ ] Parent card shows completion progress
- [ ] Subtasks cannot have their own subtasks (one level deep)

---

## EPIC-9: Comments

### US-9.1 Add comment to card

**As a** board member **I want** to add a comment to a card **so that** I can
collaborate asynchronously. **SP**: 2 **Priority**: Must **Deps**: US-5.1
**AC**:

- Given the user is a member of the board
- When the user posts a comment with non-empty body (up to 10,000 chars)
- Then the response returns the new comment including id, body, author, and
  timestamp
- Other connected clients see the new comment in real time

---

### US-9.2 List comments for card

**As a** board member **I want** to see all comments on a card in order **so
that** I can follow the conversation. **SP**: 1 **Priority**: Must **Deps**:
US-9.1 **AC**:

- The response returns all comments for the card, oldest first
- Each comment shows the author and a human-readable timestamp

---

### US-9.3 Edit / delete own comment

**As a** board member **I want** to edit or delete my own comments **so that** I
can correct mistakes. **SP**: 2 **Priority**: Must **Deps**: US-9.1 **AC**:

- Only the author can edit or delete a comment
- Other users attempting to edit or delete receive an unauthorized error
- Deleted comments are marked deleted (hidden but recoverable) rather than
  purged

---

### US-9.4 Comments UI in card modal

**As a** user **I want** a comments section in the card detail modal **so that**
I can read and write comments inline. **SP**: 3 **Priority**: Must **Deps**:
US-9.1 **AC**:

- The card modal includes a "Comments" section
- A new-comment textarea is at the bottom
- Submitting posts the comment (and on keyboard shortcut Cmd/Ctrl+Enter)
- Existing comments list with author name, avatar, and time-ago
- Hovering the user's own comment reveals edit and delete buttons

---

### EPIC-9 exit criteria

- [ ] Can post, edit, delete comments
- [ ] Comments show author + timestamp
- [ ] Comments persist across reload

---

## EPIC-10: Mentions & Watchers

### US-10.1 Watch a card (self-add)

**As a** board member **I want** to "watch" a card **so that** I'm notified of
future updates. **SP**: 2 **Priority**: Must **Deps**: US-5.1, US-3.1 **AC**:

- Given a user who is a member of the board
- When the user watches the card
- Then the user is recorded as a watcher with the timestamp they started
  watching
- Watching again is a no-op success
- Unwatching removes the user from the watchers list

---

### US-10.2 Auto-watch on interaction

**As a** board member **I want** to be auto-added as a watcher whenever I
interact with a card **so that** I stay in the loop. **SP**: 1 **Priority**:
Must **Deps**: US-10.1 **AC**:

- When a user posts a comment on a card, they are auto-added as a watcher
- When a user is assigned to a card, they are auto-added as a watcher

---

### US-10.3 @mention parsing in comments

**As a** board member **I want** typing `@username` in a comment to ping that
person **so that** collaboration is direct. **SP**: 3 **Priority**: Must
**Deps**: US-9.1 **AC**:

- When a comment containing `@username` is saved, the system parses out the
  mentions
- For each mentioned user who is a member of the board, they are auto-added as a
  watcher
- A mention event is emitted for each mentioned user so notifications can be
  dispatched later

---

### US-10.4 @mention autocomplete in textarea

**As a** user **I want** an autocomplete dropdown when I type `@` in the comment
textarea **so that** I can mention people quickly without typos. **SP**: 3
**Priority**: Must **Deps**: US-9.1 **AC**:

- Typing `@` opens a dropdown of board members
- The dropdown filters as the user types more characters
- Selection (via arrow keys or click) inserts the chosen username at the cursor
- In the comment display, mentions render as styled chips

---

### US-10.5 Watchers list in card modal

**As a** user **I want** to see who is watching a card and toggle my own watch
status **so that** I can manage my notifications. **SP**: 2 **Priority**: Must
**Deps**: US-10.1 **AC**:

- The card modal has a "Watchers" section showing avatars of current watchers
- A "Watch" / "Unwatch" toggle button is visible at the top of the modal
- Clicking the button toggles the current user's watch status

---

### EPIC-10 exit criteria

- [ ] Can watch/unwatch a card
- [ ] @mentions in comments parse and add mentioned users as watchers
- [ ] Commenting or assigning auto-watches

---

## EPIC-11: Activity Log

### US-11.1 Activity log data model

**As a** developer **I want** an activity log table **so that** every important
change is recorded. **SP**: 1 **Priority**: Must **Deps**: US-3.1 **AC**:

- An activity log table records: which board it belongs to, which user did it,
  what kind of action, the subject (type + id), the old/new values as structured
  data, and when it happened
- The table is indexed by board and creation time

---

### US-11.2 Auto-write activity entries

**As a** developer **I want** every meaningful mutation to write an activity
entry **so that** the audit trail is complete. **SP**: 3 **Priority**: Must
**Deps**: US-11.1 **AC**:

- The system writes activity entries for: board create/rename/archive, list
  create/rename/move/delete, card create/update/move/delete, comment create,
  label create, member added
- Entries include structured properties such as `{from_position, to_position}`
  for moves

---

### US-11.3 Board activity feed

**As a** board member **I want** to see the activity feed for a board **so
that** I know what's been happening. **SP**: 2 **Priority**: Must **Deps**:
US-11.2 **AC**:

- The activity feed returns paginated entries for the board, newest first
- Each entry includes the user who performed the action

---

### US-11.4 Activity feed UI

**As a** user **I want** to see activity entries rendered as readable sentences
**so that** I can scan the history quickly. **SP**: 3 **Priority**: Must
**Deps**: US-11.3 **AC**:

- Inside the card modal, an "Activity" tab shows activity entries for this card
- Optionally, a dedicated page shows board-wide activity
- Entries render as "Alice moved card X from Todo to Doing" with relative time

---

### EPIC-11 exit criteria

- [ ] Every mutation writes an activity entry
- [ ] Card modal shows activity history
- [ ] Board-level activity page works (paginated)

---

## EPIC-12: Quick Filters

### US-12.1 Quick filter data model

**As a** developer **I want** saved quick filters per user per board **so that**
each user can customize their view. **SP**: 1 **Priority**: Must **Deps**:
US-3.1 **AC**:

- A quick filter table records: who created it, which board it's for, the
  filter's display name, and a structured filter configuration
- The filter configuration can hold any combination of: assignee, labels,
  priorities, due-before / due-after, and overdue-only

---

### US-12.2 Save / list / delete quick filter

**As a** user **I want** to save, view, and delete my quick filters **so that**
I can curate my personal filter set. **SP**: 2 **Priority**: Must **Deps**:
US-12.1 **AC**:

- I can save a new filter with a name and configuration
- I can list all my saved filters for a board
- I can delete a filter I created

---

### US-12.3 Quick filter chips on board UI

**As a** user **I want** filter chips at the top of the board **so that** I can
narrow the visible cards with one click. **SP**: 5 **Priority**: Must **Deps**:
US-12.2 **AC**:

- A horizontal chip row appears at the top of the board
- A "+ New filter" button opens a form for saving a new filter
- Each chip is toggleable; active chips filter the board
- Multiple active filters combine with AND

---

### US-12.4 Default quick filters

**As a** user **I want** two sensible preset filters available out of the box
**so that** I can be productive immediately. **SP**: 1 **Priority**: Must
**Deps**: US-12.3 **AC**:

- On first visit to a board, two preset filters are visible:
  - "Only my cards" — filters to cards where I am the assignee
  - "Recently updated" — filters to cards updated within the last 7 days

---

### EPIC-12 exit criteria

- [ ] Can save, toggle, delete filters
- [ ] "Only my cards" preset exists on every board
- [ ] Filters actually narrow visible cards

---

## EPIC-13: Permissions & Roles

The personas from `requirements.md` (Admin, Project Manager, Member) are an
additive layer above this Epic. They map as: **Project Manager** = a board
`owner`/`admin`; **Member** = a board `member`; **Admin** = workspace-level
capability (out of MVP schema scope — see PUKU.md §Roles & personas and
`docs/02-features-schema-api.md` for the schema callout). No existing story is
replaced by this mapping.

### US-13.1 Confirm board-roles approach (no Spatie)

**As a** developer **I want** the board-roles approach locked in **so that**
authorization is implemented with Laravel Policies + a plain role column, not a
third-party permissions package. **SP**: 1 **Priority**: Must **Deps**: US-1.2
**AC**:

- No third-party roles/permissions package is installed (Spatie is intentionally
  not added)
- A `role VARCHAR(20)` column on `board_members` will hold one of
  `owner|admin|member|viewer`
- Authorization uses Laravel Policies (`BoardPolicy`, `CardPolicy`, …), not
  package-level abilities

---

### US-13.2 Board roles + membership data model

**As a** developer **I want** a board-membership table **so that** each user has
exactly one role per board. **SP**: 2 **Priority**: Must **Deps**: US-3.1
**AC**:

- A membership table records which users belong to which boards, and their role
- A user can have only one role per board (uniqueness enforced)
- Roles are one of: `owner`, `admin`, `member`, `viewer`
- When a board is created, the creator is auto-added as owner

---

### US-13.3 Invite member to board

**As a** board owner/admin **I want** to invite a user to the board by email
**so that** they can collaborate. **SP**: 3 **Priority**: Must **Deps**: US-13.2
**AC**:

- Given an existing user's email and a role
- When the owner/admin invites them
- Then the user is added to the board with the given role (if they have an
  account)
- If no user with that email exists, the system returns a "user not found" error
  (no pending-invitations table for MVP)
- A non-owner/admin attempting to invite receives an unauthorized error

---

### US-13.4 Change member role

**As a** board owner **I want** to change a member's role **so that** I can
adjust their access. **SP**: 1 **Priority**: Must **Deps**: US-13.3 **AC**:

- Given a board member and a new role
- When the owner changes the role
- Then the membership record is updated
- The last remaining owner cannot be demoted (a guard prevents the board from
  becoming owner-less)
- A non-owner attempting to change roles receives an unauthorized error

---

### US-13.5 Remove member

**As a** board owner/admin **I want** to remove a member from the board **so
that** I can revoke their access. **SP**: 1 **Priority**: Must **Deps**: US-13.3
**AC**:

- Given a board member
- When the owner/admin removes them
- Then the membership record is removed
- The last remaining owner cannot be removed
- All of the removed user's assignments on cards in this board are also cleared
- A non-owner/admin attempting to remove receives an unauthorized error

---

### US-13.6 Authorization policies

**As a** developer **I want** authorization policies on every action **so that**
the system enforces roles consistently. **SP**: 3 **Priority**: Must **Deps**:
US-13.2 **AC**:

- Viewing a board: any member including viewer
- Updating a board: owner or admin
- Deleting a board: owner only
- Updating a card: owner, admin, or member (not viewer)
- Every controller invokes an authorization check before performing the action

---

### US-13.7 Frontend members panel + role enforcement

**As a** user **I want** a members panel on the board and role-based UI **so
that** I can manage access and see role-appropriate controls. **SP**: 3
**Priority**: Must **Deps**: US-13.3, US-13.6 **AC**:

- The board page includes a members section listing each member with avatar,
  name, and role badge
- Owners can change roles and remove members
- Viewers see cards but cannot drag, edit, or comment; edit controls are hidden

---

### EPIC-13 exit criteria

- [ ] Owner can invite, promote, demote, remove members
- [ ] Viewer role is read-only in UI
- [ ] Backend rejects unauthorized actions
- [ ] Persona mapping (Admin / PM / Member) is documented in code comments or
      `docs/02-features-schema-api.md`

---

## EPIC-14: Real-Time Sync

### US-14.1 Install WebSocket server

**As a** developer **I want** a self-hosted WebSocket server installed **so
that** I can broadcast events. **SP**: 2 **Priority**: Must **Deps**: US-1.2
**AC**:

- The first-party WebSocket server package is installed
- The configuration file is published
- The server starts via its command and logs a successful startup message

---

### US-14.2 Authorize private board channel

**As a** developer **I want** board channels to be private and authorized **so
that** only members can subscribe. **SP**: 1 **Priority**: Must **Deps**:
US-14.1, US-13.2 **AC**:

- Subscribing to a board's channel returns success only if the user is the owner
  or a member
- Non-members are rejected with an unauthorized error

---

### US-14.3 Broadcast events

**As a** developer **I want** meaningful state changes to broadcast **so that**
connected clients can react in real time. **SP**: 3 **Priority**: Must **Deps**:
US-14.1 **AC**:

- Card events broadcast on the board's private channel: created, updated, moved,
  deleted
- List events broadcast: created, updated, moved, deleted
- Comment and label events broadcast as appropriate
- Events are emitted from the controllers/services after persistence succeeds

---

### US-14.4 Frontend real-time client setup

**As a** developer **I want** the frontend's real-time client configured **so
that** pages can subscribe to board channels. **SP**: 2 **Priority**: Must
**Deps**: US-14.1, US-1.4 **AC**:

- The real-time client libraries are installed
- The client is configured to connect to the local WebSocket server
- Configuration uses environment variables for host/port/scheme

---

### US-14.5 useRealtimeBoard hook

**As a** developer **I want** a hook that subscribes to board events and
invalidates the local cache **so that** UI updates automatically. **SP**: 3
**Priority**: Must **Deps**: US-14.3, US-14.4 **AC**:

- The hook subscribes to the board's private channel on mount
- On any event, the board's cached data is invalidated so the UI refetches
- The subscription is torn down on unmount

---

### US-14.6 Real-time verification

**As a** developer **I want** to verify real-time behavior end-to-end **so
that** I know the wiring works. **SP**: 2 **Priority**: Must **Deps**: US-14.5
**AC**:

- Two browser windows open on the same board
- Dragging a card in window A makes the move visible in window B within 1 second
- Adding a comment in window A makes it appear in window B
- Deleting a card in window A removes it from window B

---

### EPIC-14 exit criteria

- [ ] WebSocket server runs locally
- [ ] Card moves broadcast and propagate to peers
- [ ] No memory leaks (channels cleaned up on unmount)

---

## EPIC-15: All Cards View (Three-Pane)

### US-15.1 Cross-board search

**As a** user **I want** to search cards across all my boards **so that** I can
find anything quickly. **SP**: 3 **Priority**: Must **Deps**: US-5.1 **AC**:

- The search endpoint accepts any combination of: assignee, labels, priorities,
  due-before, due-after, overdue-only, board, and free-text query
- Filters are combined with AND
- Free-text matches against title and description
- Default page size 50, max 200
- Returns paginated results efficiently (no N+1 pattern)

---

### US-15.2 /cards page route

**As a** user **I want** a dedicated `/cards` route **so that** I can access the
search view from the main navigation. **SP**: 1 **Priority**: Must **Deps**:
US-1.4, US-2.7 **AC**:

- A new page exists at `/cards`
- The page is reachable from the main navigation

---

### US-15.3 Three-pane layout

**As a** user **I want** a three-pane search interface **so that** I can filter,
browse, and inspect cards simultaneously. **SP**: 5 **Priority**: Must **Deps**:
US-15.1, US-15.2 **AC**:

- Left pane: filters (~250px wide)
- Middle pane: card list (~400px wide)
- Right pane: card detail (fills remaining space)
- All three panes scroll independently
- On mobile, the layout collapses to a single pane with back navigation

---

### US-15.4 Filter sidebar

**As a** user **I want** a filter sidebar **so that** I can narrow the search by
common criteria. **SP**: 3 **Priority**: Must **Deps**: US-15.3 **AC**:

- Sections: Assignee, Labels, Priority, Due date, Board
- Each section is collapsible
- Selecting a filter chip adds it to the active filter set
- Active filters appear as removable chips above the card list

---

### US-15.5 Search input + debouncing

**As a** user **I want** a search input that updates the result list as I type
**so that** results feel instant. **SP**: 2 **Priority**: Must **Deps**: US-15.3
**AC**:

- A search input at the top of the middle pane
- The query is debounced (~300ms) before firing the search
- The current query is reflected in the URL so the view is shareable

---

### US-15.6 Card detail in right pane

**As a** user **I want** the right pane to show the selected card's full detail
**so that** I can inspect it without losing my place. **SP**: 3 **Priority**:
Must **Deps**: US-15.3 **AC**:

- Clicking a card in the middle pane loads its detail in the right pane
- The detail rendering is the same as the in-modal card detail

---

### EPIC-15 exit criteria

- [ ] `/cards` page works with all filters
- [ ] Search is debounced and URL-shareable
- [ ] Three-pane responsive on mobile

---

## EPIC-16: Production Hardening

### US-16.1 CORS configuration

**As a** developer **I want** CORS configured to allow the local frontend **so
that** the frontend can call the API. **SP**: 1 **Priority**: Must **Deps**:
US-1.2 **AC**:

- The CORS configuration allows the local frontend origin
- The frontend can successfully call authenticated endpoints

---

### US-16.2 Rate limiting on auth

**As a** developer **I want** rate limiting on login and registration **so
that** the API is protected from brute force. **SP**: 1 **Priority**: Must
**Deps**: US-2.1 **AC**:

- Login and registration endpoints are throttled
- Exceeding the limit returns a "too many requests" error

---

### US-16.3 Static analysis (backend)

**As a** developer **I want** static analysis configured and clean **so that**
type errors are caught early. **SP**: 2 **Priority**: Should **Deps**: US-1.2
**AC**:

- A static analysis tool is installed and configured for the backend
- Running it on the codebase reports no errors

---

### US-16.4 Docker Compose full stack

**As a** developer **I want** one Docker Compose file that runs the entire stack
**so that** onboarding is one command. **SP**: 3 **Priority**: Should **Deps**:
US-1.5 **AC**:

- `docker compose up` brings Postgres, Redis, the WebSocket server, the backend,
  and the frontend online together
- Postgres data persists across container restarts

---

### US-16.5 README + environment templates

**As a** developer **I want** a README and environment templates **so that**
anyone can onboard quickly. **SP**: 2 **Priority**: Must **Deps**: all **AC**:

- Root README explains the quickstart
- Backend `.env.example` documents every variable
- Frontend `.env.local.example` documents every variable
- Setup steps, test commands, and deployment notes are included

---

### EPIC-16 exit criteria

- [ ] One-command startup works on a fresh clone
- [ ] Static analysis is clean
- [ ] CORS doesn't block the frontend
- [ ] Rate limits return the correct error when exceeded

---

## Backlog Summary

| Epic                         | Stories        | Total SP   |
| ---------------------------- | -------------- | ---------- |
| EPIC-1 Project bootstrap     | 5              | 10         |
| EPIC-2 Auth & user mgmt      | 7              | 15         |
| EPIC-3 Boards                | 6              | 12         |
| EPIC-4 Lists                 | 5              | 9          |
| EPIC-5 Cards                 | 8              | 26         |
| EPIC-6 Labels                | 4              | 8          |
| EPIC-7 Assignees             | 3              | 5          |
| EPIC-8 Subtasks              | 5              | 9          |
| EPIC-9 Comments              | 4              | 8          |
| EPIC-10 Mentions & watchers  | 5              | 11         |
| EPIC-11 Activity log         | 4              | 9          |
| EPIC-12 Quick filters        | 4              | 9          |
| EPIC-13 Permissions & roles  | 7              | 14         |
| EPIC-14 Real-time sync       | 6              | 13         |
| EPIC-15 All Cards view       | 6              | 15         |
| EPIC-16 Production hardening | 5              | 9          |
| **Total**                    | **84 stories** | **182 SP** |

At ~5 SP/day solo, that's ~36 working days = ~7 weeks focused. Build it
story-by-story; each story has clear acceptance criteria so you know when "done"
is done.

---

## Recommended Implementation Order

```
Week 1: EPIC-1 (bootstrap)
Week 2: EPIC-2 (auth)
Week 3: EPIC-3 (boards) + start EPIC-4 (lists)
Week 4: EPIC-4 (lists) + EPIC-5 part 1 (card CRUD)
Week 5: EPIC-5 part 2 (drag-and-drop) + EPIC-6 (labels) + EPIC-7 (assignees)
Week 6: EPIC-8 (subtasks) + EPIC-9 (comments) + EPIC-10 (mentions/watchers)
Week 7: EPIC-11 (activity) + EPIC-12 (quick filters) + EPIC-13 (roles)
Week 6: EPIC-14 (real-time) — brought forward from Week 8 (see `03-roadmap.md`)
Week 8: EPIC-15 (all cards) + EPIC-16 (production)
```

If you have only evenings/weekends, double the timeline. The key is to **finish
stories, not start them** — a half-done story blocks nothing, but a half-done
epic blocks everything.
