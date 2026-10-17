# KanbanFlow — Stack & Architecture

## Stack

| Layer           | Choice                                                                 |
| --------------- | ---------------------------------------------------------------------- |
| Backend         | Laravel 12 (PHP 8.3+)                                                  |
| Auth            | Sanctum bearer tokens                                                  |
| Authz           | Laravel Policies + plain `role VARCHAR(20)` on `board_members`         |
| Database        | PostgreSQL 16                                                          |
| Real-time       | Reverb + Echo + pusher-js                                              |
| Frontend        | Next.js 15 (App Router) + React 19 + TypeScript                        |
| Styling         | **Custom CSS (default) + CSS Modules**. Tailwind installed but demoted |
| State (UI)      | Zustand                                                                |
| State (server)  | TanStack Query                                                         |
| Forms           | React Hook Form + Zod                                                  |
| Drag & drop     | @dnd-kit                                                               |
| Testing         | Pest + Vitest + RTL                                                    |
| Static analysis | Larastan + TypeScript                                                  |
| Local infra     | Docker Compose (one root-level file, 4 services — see D3)              |

> **Custom CSS is the default.** Tailwind is installed but demoted to optional
> helper. See **Styling** section below and
> `05-task-and-skill-task-templates.md` §CSS Strategy Lock-In for details.

## Architecture

```
┌─────────────────────────────┐         ┌─────────────────────────────┐
│ Next.js 15 (App Router)     │         │ Laravel 12 (API)            │
│ - TanStack Query            │  HTTPS  │ - Sanctum bearer            │
│ - Zustand                   │ ◀────▶  │ - Eloquent + Resources      │
│ - @dnd-kit                  │  JSON   │ - FormRequest + Policies    │
│ - Laravel Echo              │         │ - Reverb (WebSocket)        │
│ - Custom CSS + CSS Modules  │  WSS    │ - Plain role + Policies     │
└─────────────────────────────┘         └─────────────────────────────┘
                                                  │
                                          ┌───────┴────────┐
                                          │ PostgreSQL 16  │
                                          │ Redis          │
                                          └────────────────┘
```

**Request flow**: drag card → Zustand updates locally (optimistic) → TanStack
Query `mutate()` → `PATCH /api/v1/cards/{id}/move` → controller validates +
authorizes + updates + fires `CardMoved` → Reverb broadcasts to `board.{id}` →
Echo on peers invalidates their query.

## Backend setup

```bash
composer create-project laravel/laravel:^12 api
cd api
php artisan install:api
composer require laravel/reverb
php artisan reverb:install
composer require -D pestphp/pest pestphp/pest-plugin-laravel larastan/larastan
```

> Folder name is **`api/`** (not `kanbanflow-api/`). Spatie Permission is
> **not** installed — we use a plain `role VARCHAR(20)` column on
> `board_members` plus Laravel Policies (see Styling / Authz above).

### Folder structure

```
api/
├── app/
│   ├── Http/
│   │   ├── Controllers/Api/V1/    # thin controllers
│   │   ├── Requests/              # FormRequest validation
│   │   └── Resources/             # API Resources (response shaping)
│   ├── Models/                    # User, Board, BoardList, Card,
│   │                              #   Label, Comment, BoardMember, ActivityLog
│   ├── Policies/                  # BoardPolicy, CardPolicy, ListPolicy
│   ├── Services/                  # BoardService, CardMoveService, ActivityLogger
│   ├── Events/                    # broadcastable Card/List/Comment events
│   └── Broadcasting/              # channel auth
├── database/migrations/
├── routes/api.php                 # /api/v1/*
└── tests/{Feature,Unit}/
```

### Conventions

- API versioning: `/api/v1/...` from day one.
- Thin controllers → FormRequest + Service → API Resource.
- Policies for ownership (`$user->can('update', $board)`).
- Roles: `owner|admin|member|viewer` stored as plain `VARCHAR(20)` on
  `board_members.role`. No Spatie.
- Broadcast events fired automatically after persistence.

### Sanctum (bearer)

```php
// config/sanctum.php
'stateful' => [],

// User model
use HasApiTokens;

// routes/api.php
Route::middleware('auth:sanctum')->prefix('v1')->group(function () {
    Route::apiResource('boards', BoardController::class);
});
```

### Reverb

```env
BROADCAST_CONNECTION=reverb
REVERB_APP_ID=kanbanflow
REVERB_APP_KEY=local-key
REVERB_APP_SECRET=local-secret
REVERB_HOST=localhost
REVERB_PORT=8080
REVERB_SCHEME=http
```

```php
// app/Events/CardMoved.php
class CardMoved implements ShouldBroadcast {
    public function __construct(public Card $card) {}
    public function broadcastOn(): PrivateChannel {
        return new PrivateChannel('board.' . $this->card->list->board_id);
    }
}
```

## Frontend setup

```bash
pnpm create next-app@latest web --ts --tailwind --app --eslint --src-dir --import-alias "@/*"
cd web
pnpm i @tanstack/react-query @tanstack/react-query-devtools zustand \
      react-hook-form @hookform/resolvers zod \
      @dnd-kit/core @dnd-kit/sortable @dnd-kit/utilities \
      axios laravel-echo pusher-js
pnpm i -D vitest @testing-library/react @testing-library/jest-dom jsdom
```

> Folder name is **`web/`** (not `kanbanflow-web/`). Package manager is
> **`pnpm`**. Tailwind is installed but demoted — see "Styling" below and
> `05-task-and-skill-task-templates.md` §CSS Strategy Lock-In.

### Folder structure

```
web/
├── src/
│   ├── app/
│   │   ├── (auth)/login,register
│   │   └── (app)/boards/[boardId], settings
│   ├── components/{board, ui, auth}/
│   ├── lib/api/{client, boards, cards, lists}.ts
│   ├── lib/echo.ts
│   ├── stores/{useBoardStore, useUiStore}.ts
│   ├── hooks/{useBoard, useMoveCard, useRealtimeBoard}.ts
│   ├── schemas/{auth, card}.ts     # Zod
│   ├── types/api.ts
│   └── providers/{QueryProvider, AuthProvider}.tsx
├── src/app/globals.css             # @layer reset, base, components, utilities
├── src/app/layout.tsx               # imports globals.css once at the root
├── scripts/css-leaderboard.sh      # Tailwind budget lint (D11)
└── .css-leaderboard-budget.json    # per-Phase Tailwind budgets
```

### Server vs Client Components

- **RSC** (default): boards list, settings, marketing.
- **Client (`"use client"`)**: drag context, modals, forms.

Pattern: `page.tsx` is RSC, fetches board, renders
`<BoardShell initialBoard={board}>` (Client Component owning drag context +
Zustand).

```tsx
// app/(app)/boards/[boardId]/page.tsx (RSC)
const board = await fetchBoard(params.boardId);
return <BoardShell initialBoard={board} />;

// components/board/BoardShell.tsx (Client)
("use client");
export function BoardShell({ initialBoard }) {
  const setBoard = useBoardStore((s) => s.setBoard);
  useEffect(() => setBoard(initialBoard), [initialBoard]);
  useRealtimeBoard(initialBoard.id);
  return <DndContext onDragEnd={handleDragEnd}>...</DndContext>;
}
```

### Real-time wire-up

```ts
// lib/echo.ts
import Echo from "laravel-echo";
import Pusher from "pusher-js";
window.Pusher = Pusher;

export const echo = new Echo({
  broadcaster: "reverb",
  key: process.env.NEXT_PUBLIC_REVERB_APP_KEY!,
  wsHost: process.env.NEXT_PUBLIC_REVERB_HOST!,
  wsPort: Number(process.env.NEXT_PUBLIC_REVERB_PORT!),
  forceTLS: false,
  enabledTransports: ["ws", "wss"],
});
```

```ts
// hooks/useRealtimeBoard.ts
useEffect(() => {
  const channel = echo
    .private(`board.${boardId}`)
    .listen("CardMoved", () =>
      queryClient.invalidateQueries({ queryKey: ["board", boardId] }),
    );
  return () => {
    echo.leave(`board.${boardId}`);
  };
}, [boardId]);
```

## Docker Compose

```yaml
# docker-compose.yml  (repo root — NOT inside api/)
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: kanbanflow
      POSTGRES_USER: kanban
      POSTGRES_PASSWORD: kanban
    ports: ["5432:5432"]
    volumes: ["pgdata:/var/lib/postgresql/data"]

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

  # Reverb + app come online in EPIC-9 / EPIC-16.
  # EPIC-1 ships the postgres + redis subset only.
  # reverb:
  #   build: ./api
  #   command: php artisan reverb:start
  #   ports: ["8080:8080"]
  #   env_file: ./api/.env
  #   depends_on: [postgres, redis]
  #
  # app:
  #   build: ./api
  #   command: php artisan serve --host=0.0.0.0
  #   ports: ["8000:8000"]
  #   env_file: ./api/.env
  #   depends_on: [postgres, redis, reverb]

volumes:
  pgdata:
```

Next.js runs separately via `pnpm dev` on port 3000.

## Environment variables

```env
# api/.env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=kanbanflow
DB_USERNAME=kanban
DB_PASSWORD=kanban
BROADCAST_CONNECTION=reverb
QUEUE_CONNECTION=redis
CACHE_STORE=redis
REVERB_APP_ID=kanbanflow
REVERB_APP_KEY=local-key
REVERB_APP_SECRET=local-secret
REVERB_HOST=localhost
REVERB_PORT=8080
REVERB_SCHEME=http
SANCTUM_STATEFUL_DOMAINS=
```

```env
# web/.env.local
NEXT_PUBLIC_API_URL=http://localhost:8000/api/v1
NEXT_PUBLIC_REVERB_APP_KEY=local-key
NEXT_PUBLIC_REVERB_HOST=localhost
NEXT_PUBLIC_REVERB_PORT=8080
NEXT_PUBLIC_REVERB_SCHEME=http
```

> One `KEY=value` per line.

## Styling

**Tailwind is installed but demoted.** Custom CSS is the default for layout,
theming, animations, state changes, design tokens, and component styling.
Tailwind is allowed only for trivial spacing (`mt-2`, `px-4`), color helpers
(`text-gray-500`), and pre-built accessible primitives (`sr-only`).

### CSS Strategy

- **One `app/globals.css` per Next.js app**, declared once in `app/layout.tsx`.
- **Four cascade layers** in declared order:
  `@layer reset, base, components, utilities;`. Unlayered styles always win
  against layered styles — never write styles outside these four layers.
- **Design tokens as CSS custom properties** inside
  `@layer base { :root { … } }` (e.g., `--color-bg`, `--color-text`,
  `--space-md`, `--radius-sm`, `--shadow-md`, `--duration-base`, `--ease-out`).
- **Component styles via CSS Modules** (`Foo.module.css`). Named exports.
- **`@apply` is banned in component code** — it ties you to Tailwind config +
  PostCSS and defeats the custom-CSS default.
- **Logical properties from day one** (`margin-inline`, `padding-block`) so RTL
  support is free later.
- **One global `@media (prefers-reduced-motion: reduce)` block** at the bottom
  of `globals.css` that disables transitions and animations globally; every
  Interaction Choreography in a Task respects this convention.
- **`web/scripts/css-leaderboard.sh`** is a CI-friendly grep check that fails
  the build when utility-class usage in `web/components/**` and `web/app/**`
  exceeds the per-Phase budget in `web/.css-leaderboard-budget.json`. The budget
  starts at 0 (Phase 1 has no `web/`) and grows to 56 in Phase 8. **The script
  is shipped in T-1.4 (Step 6) — it does not exist on disk yet.** Until then,
  the per-Phase budgets below are the binding contract.

### Next.js + CSS Constraints

The Next.js App Router has opinions about CSS that the developer must
internalize:

- **CSS Modules work in Server Components.**
  `import styles from './foo.module.css'` resolves at build time and the
  resulting CSS is shipped as a separate stylesheet.
- **Global CSS via `app/layout.tsx`.** Imported once at the root. Order matters:
  `@layer reset` blocks before component CSS.
- **CSS-in-JS forces `'use client'`.** Avoid styled-components / emotion /
  vanilla-extract unless you're willing to make that boundary explicit.
- **FOUC risk:** late-injected design tokens cause a flash of unstyled content.
  Solution: declare tokens in `app/globals.css` inside
  `@layer base { :root { --color-bg: … } }`. They're resolved at first paint.
- **z-index stacking contexts** are created by `transform`, `will-change`,
  `filter`, `mix-blend-mode`, `isolation: isolate`. Stacking contexts trap
  z-indexes — a `transform: translateZ(0)` on a parent will hide a modal
  `z-index: 9999`. Document any place we use these.
- **`prefers-reduced-motion`** must be respected in every Interaction
  Choreography. Project convention: ship a single global
  `@media (prefers-reduced-motion: reduce)` block at the bottom of
  `app/globals.css` that sets `transition-duration: 0.01ms !important` and
  `animation-duration: 0.01ms !important` on `*, *::before, *::after`. This is
  intentionally aggressive — it kills **all** motion including shadow fades —
  because the cost of accidentally ignoring a user's OS-level motion preference
  is higher than the cost of losing a decorative fade. If you need to preserve a
  specific motion for accessibility (e.g., a focus-visible outline), do it via
  state change, not animation.
- **Turbopack dev caching:** Next.js 15 ships with Turbopack as the default dev
  bundler, fast but aggressive about caching — if `globals.css` looks unchanged
  when you edit, run `rm -rf .next` and restart.

### When Tailwind is allowed (D11)

Default to custom CSS for: layout, theming, animations, state changes, design
tokens, component styling, anything you'd write a `Learn` paragraph about.
Tailwind is allowed for trivial spacing, color helpers, and pre-built accessible
primitives. **CSS-only Monday**: one Task per Phase must be implemented with
zero utility classes. **CSS-leaderboard lint** fails the build when Tailwind
usage exceeds the per-Phase budget.

## Dependencies

**Composer**: `laravel/sanctum`, `laravel/reverb`, (dev) `pestphp/pest`,
`pestphp/pest-plugin-laravel`, `larastan/larastan`, `laravel/pint`.

> Spatie Permission is **not** installed. We use a plain `role VARCHAR(20)`
> column on `board_members` plus Laravel Policies (see Styling / Authz above and
> the templates).

**pnpm** (this project uses pnpm, not npm): `@tanstack/react-query`+devtools,
`zustand`, `react-hook-form`+`@hookform/resolvers`+`zod`,
`@dnd-kit/{core,sortable,utilities}`, `laravel-echo`+`pusher-js`, `axios`,
`tailwindcss` (installed but demoted), (dev) `vitest`+`@testing-library/react`.
