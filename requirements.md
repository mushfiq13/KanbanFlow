# KanbanFlow - v1 Requirements

## Problem

For a small group of people where they do the shared work - need a platform to
sync their work with each other.

PMs and developers write down what needs doing — including user stories, tasks,
and bugs and developers work on that description. Each member needs to see the
same state of the work, updated in real time.

A way to sync is to use Excel but this is harder to maintain.

**A shared Kanban Board:** syncs the work in real-time, and keeps an activity
log of who did what - for future references.

## FRs - Functional Requirements

| #     | Requirement                                                              | How To Test It                                                                                                                                                                                                                     |
| ----- | ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| FR-1  | Users can register using Email/Password                                  | Send `/api/v1/register [POST]`. If succeeded, then return a body with `{user, token}` with `201 status code`; otherwise return `422` validation error if the email is already taken and/or the password is less than 8 characters. |
| FR-2  | Users can log in via Email/Password                                      | Send `/api/v1/login [POST]`. If succeeded, then return `{user, token}` with `200` status code; otherwise return `401` error if the email or password is not found.                                                                 |
| FR-3  | Users can log out from a logged-in account                               | Send `/api/v1/logout [POST]`. It shall return `204` status code. In the website, remove the token from client storage and reset the auth state.                                                                                    |
| FR-4  | Users can soft-delete their account.                                     | Soft-deleted users cannot access their account, but their histories are visible to other board members.                                                                                                                            |
| FR-5  | The system shall have three roles: admin, project manager, and member.   | Create three users, each assigned to one role, and verify the role list.                                                                                                                                                           |
| FR-6  | The system shall have a workspace per organization.                      | For the first time, a user can log in as the org owner. Then he/she can add other users of his/her org under this workspace.                                                                                                       |
| FR-7  | Admin can create boards and assign one project manager per board.        | Create a board as an admin user, assign a member as the project manager of that board.                                                                                                                                             |
| FR-8  | Board members can create individual cards.                               | Ensure an authorized board member can create a card, then write down a description. All board members can see created cards of the board.                                                                                          |
| FR-9  | The system shall allow creating custom lists.                            | Create several lists such as ToDo, InProgress, CodeReview, InQA, Closed. Board members shall be allowed to drag-and-drop a card from one list to another.                                                                          |
| FR-10 | Project manager can assign any users as project members to a board.      | A user assigned as project manager is able to add board members.                                                                                                                                                                   |
| FR-11 | Admin can see all boards; user can see only boards they are a member of. | Admin user must see all created boards.                                                                                                                                                                                            |

## NFRs - Non-Functional Requirements

| #     | Requirement | Metric | Target | Load Condition | Measurement Method |
| ----- | ----------- | ------ | ------ | -------------- | ------------------ |
| NFR-1 |             |        |        |                |                    |
