# KanbanFlow — Context

## What

Trello-like kanban web app (boards, lists, cards, drag-and-drop, real-time).
Learning project for a .NET/C# engineer to practice React/Next.js and Laravel.

## Why Trello

Trello started as a 5-item to-do list at a Fog Creek hackathon (2010), launched
2011, acquired by Atlassian in 2017 for **$425M**, crossed 50M users. Its MVP
was minimal:

- **Boards → Lists → Cards** (three nouns)
- **Drag-and-drop** (one verb)
- **Real-time collaboration**
- **Free tier**

The genius was a frictionless translation of sticky notes — not novel features.
Trello won by being small, not big.

## Trello vs alternatives

| Product  | Positioning               | Lesson                     |
| -------- | ------------------------- | -------------------------- |
| Trello   | Kanban-only, simple, free | Our reference              |
| Asana    | Lists + timelines + goals | Kanban is not the focus    |
| ClickUp  | "Replace all apps"        | Anti-pattern — scope creep |
| Notion   | Docs + databases          | Different mental model     |
| Jira     | Engineering-heavy         | Too much ceremony          |
| Bitrix24 | Enterprise CRM suite      | Not same category          |

## Confirmed decisions

1. **Auth**: Laravel Sanctum bearer tokens (avoids CORS pain in dev).
2. **Real-time**: In MVP — Laravel Reverb + Echo.
3. **Database**: PostgreSQL from day one.
4. **Plan layout**: Split into 5 product docs (00–04) plus the curriculum
   templates (`05-task-and-skill-task-templates.md` and `06-css-curriculum.md`).

## Project layout

```
KanbanFlow/
├── api/                    # Laravel 12 backend (created in EPIC-1, T-1.2)
├── web/                    # Next.js 15 frontend (created in EPIC-1, T-1.4)
├── docker-compose.yml      # root-level; Postgres + Redis + Reverb + app
├── docs/                   # product spec + curriculum
├── .gitignore
├── PUKU.md                 # this file
└── README.md               # quickstart
```

## Document map

| File                                  | Contents                                           |
| ------------------------------------- | -------------------------------------------------- |
| `00-context.md`                       | This file                                          |
| `01-stack.md`                         | Tech stack, folder structures, infra               |
| `02-features-schema-api.md`           | Features, DB schema, API                           |
| `03-roadmap.md`                       | Mental model map, milestones, verification         |
| `04-user-stories.md`                  | 16 epics / 84 user stories with Given/When/Then AC |
| `05-task-and-skill-task-templates.md` | Task + Skill Task templates + worked examples      |
| `06-css-curriculum.md`                | CSS Topic Map + Deep Lessons + 3-part Mastery Test |

## Out of scope

Mobile native, OAuth, billing, power-ups, automation rules, calendar/timeline,
PWA/offline. Real product features but not the point of this learning project.
