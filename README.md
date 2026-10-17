# KanbanFlow

Trello-like kanban application built as a **learning project** to practice
**React** and **Laravel**.

## What is this?

KanbanFlow is a single-page web app for organizing work on boards, lists, and
cards — with drag-and-drop, real-time multi-user sync, and member roles.

It is **not** a clone of Trello, Jira, Asana, or ClickUp.

## Repository layout

This is a **monorepo** with two siblings that grow into two deployable
artifacts:

```
KanbanFlow/
├── docs/                 ← product spec, user stories, curriculum
├── requirements.md        cross-cutting rules
├── api/
├── web/
├── docker-compose.yml
├── .gitignore
├── PUKU.md
└── README.md
```

## Tech stack

| Layer           | Choice                                                                                                                                                                                       |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Backend         | Laravel 12 (PHP 8.3+)                                                                                                                                                                        |
| Auth            | Sanctum bearer tokens                                                                                                                                                                        |
| Authz           | Laravel Policies + plain `role VARCHAR(20)` on `board_members`                                                                                                                               |
| Data access     | Eloquent ORM only, accessed via **repository classes** injected into controllers (no raw SQL, no `DB::` query builder in app code)                                                           |
| Database        | PostgreSQL 16                                                                                                                                                                                |
| Real-time       | Laravel Reverb + Laravel Echo + pusher-js                                                                                                                                                    |
| Frontend        | **Vite + React 19 + TypeScript** (no Next.js)                                                                                                                                                |
| Routing         | TanStack Router (type-safe)                                                                                                                                                                  |
| State (UI)      | Zustand                                                                                                                                                                                      |
| State (server)  | TanStack Query                                                                                                                                                                               |
| Forms           | React Hook Form + Zod                                                                                                                                                                        |
| Drag & drop     | @dnd-kit                                                                                                                                                                                     |
| Styling         | **Custom CSS only** — CSS Modules + a single `src/styles/globals.css` with four cascade layers and design tokens. **No Tailwind.** See `docs/06-css-curriculum.md` for the binding contract. |
| Testing         | Pest + Vitest + React Testing Library                                                                                                                                                        |
| Static analysis | Larastan + TypeScript                                                                                                                                                                        |
| Local infra     | Docker Compose (Postgres + Redis + Reverb + app)                                                                                                                                             |

## License

TBD. Personal learning project; not currently intended for redistribution.

# KanbanFlow
