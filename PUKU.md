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

## Documentation

All planning artifacts live under `docs/`:

| File                                       | What's in it                                                    |
| ------------------------------------------ | --------------------------------------------------------------- |
| `docs/00-context.md`                       | Why Trello, MVP scope, confirmed decisions                      |
| `docs/01-stack.md`                         | Tech stack, folder structures, infrastructure                   |
| `docs/02-features-schema-api.md`           | Features, DB schema, REST API                                   |
| `docs/03-roadmap.md`                       | .NET → Laravel/React mental model, 8-week milestones            |
| `docs/04-user-stories.md`                  | 84 PM-language user stories across 16 epics                     |
| `docs/05-task-and-skill-task-templates.md` | Task + Skill Task templates + T-1.2 / T-1.4 worked examples     |
| `docs/06-css-curriculum.md`                | CSS Topic Map + Deep Lessons + 3-part Post-CSS Mastery Test     |
| `requirements.md`                          | v1 scope: 12 Tier-1 features, cross-cutting rules, Out-of-scope |

**Read in order**: 00 → 01 → 02 → 03 → 04 (then 05 + 06 when you're ready to
start Tasks).

## Tech stack (planned)

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

## Development workflow

This repo is in the **planning phase**. No code has been written yet.

To build KanbanFlow:

1. Read `docs/00-context.md` → understand the why.
2. Read `docs/01-stack.md` → understand the architecture.
3. Read `docs/02-features-schema-api.md` → understand the data + API.
4. Read `docs/03-roadmap.md` → understand the 8-week sequence.
5. Read `docs/04-user-stories.md` → pick a story and implement it.
6. Mark the corresponding epic task as `in_progress` in the project tracker,
   then `completed` when the epic's exit criteria are met.

Each user story has Given/When/Then acceptance criteria — "done" means every AC
passes.

## Why this exists

KanbanFlow is small enough to finish and rich enough to exercise:

- REST API design and versioning
- Authentication with bearer tokens
- Real-time WebSockets end-to-end
- Drag-and-drop UI patterns
- Authorization (policies + roles)
- Soft delete, audit logs, optimistic concurrency
- Containerized local dev with Docker Compose
- **Modern CSS as a first-class deliverable**: cascade layers, container
  queries, `:has()`, `@starting-style`, fluid type via `clamp()`, CSS nesting,
  `:where()`/`:is()`, `@scope`, `color-mix()`, and the `prefers-reduced-motion`
  contract — see `docs/06-css-curriculum.md`. **Custom CSS only; no Tailwind or
  other utility-class framework is installed.**

If you are reading this and thinking "I'd build it differently" — you probably
would, and that's fine. This is a learning project, not a product.

## License

TBD. Personal learning project; not currently intended for redistribution.
