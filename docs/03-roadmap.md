# KanbanFlow — Roadmap & Verification

## .NET → Laravel/React mental model

| ASP.NET / C#                            | Laravel / Next.js                                |
| --------------------------------------- | ------------------------------------------------ |
| ASP.NET Core MVC                        | Laravel MVC                                      |
| Controller                              | Controller (thin, delegates to services)         |
| `[ApiController]` + filters             | FormRequest + Middleware + Policies              |
| `IActionFilter`                         | Middleware                                       |
| `DbContext` / `DbSet<T>`                | Eloquent (Active Record — model **is** the repo) |
| LINQ                                    | Eloquent query builder                           |
| `IQueryable<T>` deferred                | Eloquent builder deferred until `->get()`        |
| DataAnnotations / FluentValidation      | `FormRequest` classes                            |
| AutoMapper / Mapster                    | API Resources                                    |
| Identity / JWT                          | Sanctum (bearer here)                            |
| `[Authorize]` / `IAuthorizationHandler` | Policies + Gates                                 |
| Hosted services                         | Jobs + Queue                                     |
| SignalR                                 | Laravel Reverb                                   |
| `dotnet ef migrations`                  | `php artisan make:migration`                     |
| xUnit / NUnit                           | Pest                                             |
| `ILogger<T>`                            | `Log::info()` or `LoggerInterface`               |
| DI via constructor                      | Same — auto-resolved via reflection              |
| Razor partials                          | React components                                 |
| DI lifetimes                            | React hooks (runtime, not container)             |
| Razor Pages / Blazor server             | Next.js RSC                                      |
| Minimal API / POST                      | Server Actions                                   |
| Controllers + Views                     | Next.js (App Router + Client Components)         |

### Biggest shift: EF Core → Eloquent

EF Core = Unit-of-Work. `await _db.SaveChangesAsync()` commits.

Eloquent = Active Record. The model persists itself:

```php
$card = Card::find(42);
$card->title = 'Updated';
$card->save(); // no SaveChanges
```

- **Loading**: `$board->load('lists.cards')` (eager) vs `$board->lists` (lazy)
  vs `$board->lists()->with('cards')->get()` (builder).
- **Change tracking**: mutate any property → `$model->save()` persists all dirty
  attrs (`isDirty()`).
- **Transactions**: `DB::transaction(fn() => ...)` ≈ `BeginTransactionAsync()`.

### C# developer pitfalls

1. PHP types are loose — use `declare(strict_types=1);` + Larastan.
2. PHP uses `null`, not `undefined`.
3. Always `===`, never `==`.
4. PHP is synchronous; use Jobs for concurrency.
5. Composer = NuGet (`composer require` updates lockfile; `dump-autoload` if
   classes missing).
6. `php artisan` = `dotnet` CLI.
7. `.env` = `appsettings.Development.json`. Never commit.
8. Composer for PHP, npm for JS — don't confuse.
9. Laravel has strong naming conventions — follow them.
10. N+1 queries are easy in Eloquent — always `with()`.
11. Never `$card->update($request->all())` — use `$fillable` or
    `$card->fill($request->validated())`.

### React mental model for C# devs

- **Components** = Razor partials (props = parameters).
- **Hooks** = DI lifetimes, but runtime-managed.
- **State** ≈ ViewModels, but lives in the render tree.
- **JSX** = HTML-in-JS.
- **`useState`** ≈ local field. **`useEffect`** ≈
  `OnInitialized`/`OnAfterRender`.
- **Context** = ambient service. Don't abuse for high-frequency state.

## Milestones (8 weeks)

Goal is learning, not finishing on schedule — prioritize finishing over the
calendar.

### Week 1 — Laravel foundations

- Install Laravel 12, `php artisan install:api`, set up Postgres.
- Migrations + models for `User`, `Board`, `BoardList`, `Card`.
- CRUD endpoints with FormRequest + API Resources.
- Pest feature tests for `BoardController`.
- **Deliverable**: `GET /api/v1/boards` returns JSON; `POST` creates a row.

### Week 2 — Auth & authorization

- Sanctum; `HasApiTokens` on User.
- `/register`, `/login`, `/logout`, `/me` returning bearer tokens.
- Plain `role VARCHAR(20)` on `board_members` (`owner|admin|member|viewer`).
  **No Spatie** — we use Laravel Policies for authorization.
- Personas (admin / project manager / member) from `requirements.md` map onto
  the board-level roles: **PM** ≈ `owner|admin`, **Member** ≈ `member`,
  **Admin** ≈ workspace-level capability tracked out-of-band for MVP (see
  PUKU.md §Roles & personas).
- `BoardPolicy`, `CardPolicy` in controllers.
- `board_members` table + invite endpoint.
- **Deliverable**: Invite a member; viewer role can't edit.

### Week 3 — Next.js foundations

- `create-next-app` with App Router + TS + Tailwind.
- `/login`, `/register` with React Hook Form + Zod.
- `lib/api/client.ts` — axios with bearer interceptor from localStorage.
- TanStack Query provider.
- `/boards` page using `useQuery`.
- **Deliverable**: Log in, see boards list from Laravel.

### Week 4 — Kanban UI (drag & drop)

- `@dnd-kit/{core,sortable,utilities}`.
- `/boards/[boardId]`: lists horizontal, cards vertical.
- Zustand store for in-flight drag state.
- `useMoveCard` mutation with optimistic update + rollback.
- Card detail modal: title, description, due date.
- **Deliverable**: Drag cards within/across lists; persists after refresh.

### Week 5 — Relationships & polish

- Labels, assignees, comments.
- Card modal UI polish.
- Activity log writes from `CardMoveService`.
- **Deliverable**: Card with labels, assignees, comments, audit trail.

### Week 6 — Real-time (Reverb + Echo)

- Install Laravel Reverb; configure `.env`.
- Broadcast events (`CardMoved`, `CardCreated`, ...).
- Authorize private `board.{id}` channel.
- `laravel-echo` + `pusher-js` on client.
- `useRealtimeBoard` hook subscribes + invalidates queries.
- **Deliverable**: Drag in one tab, see move in another in real time.

### Week 7 — Members & roles in UI

- Members panel; invite-by-email.
- Role badges; viewer = read-only in UI.
- Soft delete + restore (boards, lists, cards).
- Archived boards toggle in list.
- **Deliverable**: Complete multi-user experience with roles.

### Week 8 — Production prep

- `QUEUE_CONNECTION=redis`; notification jobs.
- Docker Compose orchestration.
- CORS config (`config/cors.php`) — allow `http://localhost:3000`.
- Rate limiting on auth endpoints.
- Vitest + RTL for a few components.
- Larastan in CI.
- **Deliverable**: One-command startup; deployable shape.

## Verification

### Backend smoke test (after Week 1)

```bash
docker compose up -d
php artisan migrate:fresh --seed

curl -X POST http://localhost:8000/api/v1/register \
  -H 'Content-Type: application/json' \
  -d '{"name":"Mushf","email":"m@example.com","password":"secret123","password_confirmation":"secret123"}'

TOKEN="..."   # from response

curl http://localhost:8000/api/v1/me -H "Authorization: Bearer $TOKEN"
curl -X POST http://localhost:8000/api/v1/boards \
  -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' \
  -d '{"title":"Q4 Roadmap"}'

psql -h 127.0.0.1 -U kanban kanbanflow -c 'SELECT * FROM boards;'
```

### Automated tests

```bash
./vendor/bin/pest
```

Coverage:

- Each `*Controller`: 1 happy-path + 1 auth-failure test.
- Each `*Policy`: 1 allow + 1 deny test.
- `CardMoveService`: same-list, cross-list, gap-below-1 cases.

### Frontend smoke test (after Week 3)

```bash
cd web && pnpm dev
# http://localhost:3000 → log in → /boards lists ≥ 1 board
```

### Drag verification (after Week 4)

1. Open `/boards/1` in two browser windows.
2. Drag a card from "Todo" to "Doing" in window A.
3. **Expected**: A is instant; B catches up in ~500ms.
4. Reload A — card stays in "Doing".

### Real-time verification (after Week 6)

1. Three terminals: `php artisan reverb:start`, `php artisan serve`, `pnpm dev`.
2. Same board in two browser tabs.
3. Drag in tab 1 — tab 2 updates without refresh.
4. Check Reverb logs for broadcasts.
5. DevTools → Network → WS: connection open, events arriving.

### Multi-user flow (after Week 7)

1. Register Alice + Bob.
2. Alice creates board, invites Bob as `member`.
3. Bob logs in, sees board.
4. Bob tries to rename → denied.
5. Alice promotes Bob to `admin`.
6. Bob renames → succeeds.

### Static analysis

```bash
./vendor/bin/phpstan analyse     # backend — Larastan under the hood
pnpm tsc --noEmit                 # frontend
```

Both clean before declaring a milestone done.

## MVP acceptance criteria

- [ ] Two users can register, log in, log out.
- [ ] Create a board, see in boards list, open it.
- [ ] Board shows lists (title) and cards (title, description, due date).
- [ ] Cards: create, edit (modal), delete.
- [ ] Cards drag within list and across lists; move persists.
- [ ] Lists reorder via drag; persists.
- [ ] Invite user by email; they see the board.
- [ ] Roles enforced backend (Policies) and reflected in UI.
- [ ] Cards have labels + assignees; both editable.
- [ ] Cards have comments; show author + timestamp.
- [ ] Activity log records moves, creates, edits.
- [ ] Two browsers on same board — drag in one updates the other in < 1s.
- [ ] All Pest feature tests pass; ≥ 1 RTL component test passes.
- [ ] No Larastan errors at configured level.

All boxes checked → MVP done. Anything beyond is v1.1.
