# KanbanFlow — Features, Schema, API

## Feature tiers

### MUST HAVE (MVP)

| #   | Feature                                           |
| --- | ------------------------------------------------- |
| F1  | Email/password auth (Sanctum bearer)              |
| F2  | Boards CRUD                                       |
| F3  | Lists CRUD + reorder via drag                     |
| F4  | Cards CRUD + drag within/across lists             |
| F5  | Card detail modal (title, desc, due date)         |
| F6  | Labels (per-board color tags)                     |
| F7  | Assignees (multi-user per card)                   |
| F8  | Comments                                          |
| F9  | Board members + roles (owner/admin/member/viewer) |
| F10 | Activity log                                      |
| F11 | Real-time sync (Reverb + Echo)                    |
| F12 | Versioned REST API (`/api/v1/...`)                |

### SHOULD HAVE (v1.1)

Checklists, attachments, search, filters, email notifications, soft delete +
restore, board themes, user profile+avatar, keyboard shortcuts, recent boards.

### NICE TO HAVE (later)

Card covers, board templates, automation rules, calendar/timeline views, mobile
responsive, PWA, native apps, integrations.

> **Discipline**: when tempted to add SHOULD HAVE during MVP, write it down and
> move on.

## Database schema (PostgreSQL)

### ER

```
users ─┬── boards ─── board_members ─── users
       │      │
       │      └── lists ─── cards ─── comments ─── users
       │                      │  ╲
       │                      │   ╲── card_labels ─── labels
       │                      └── activity_log
       └── personal_access_tokens (Sanctum)
```

### Tables

```sql
-- users
id, name, email (unique), email_verified_at, password, avatar_path, remember_token, timestamps

-- personal_access_tokens (Sanctum-managed)

-- boards
id, owner_id FK→users (RESTRICT), title, description, background,
archived_at, timestamps, deleted_at
INDEX (owner_id)

-- board_members
id, board_id FK→boards (CASCADE), user_id FK→users (CASCADE),
role VARCHAR(20)  -- owner|admin|member|viewer
UNIQUE (board_id, user_id)

> **Personas (additive layer).** `requirements.md` defines three personas —
> **Admin**, **Project Manager**, **Member** — at the workspace and board
> scope. They map onto this schema as: **Admin** = workspace-level capability
> (not yet modeled as a column in MVP); **Project Manager** = board-level
> `owner` or `admin` row in `board_members`; **Member** = board-level
> `member` row. The `board_members.role` column remains the authorization
> source of truth per board.

-- lists
id, board_id FK→boards (CASCADE), title, position INT,
archived_at, timestamps, deleted_at
INDEX (board_id, position)

-- cards
id, list_id FK→lists (CASCADE), title, description, position INT,
due_date, cover_color, assigned_to FK→users (SET NULL),
archived_at, timestamps, deleted_at
INDEX (list_id, position), INDEX (assigned_to)

-- labels
id, board_id FK→boards (CASCADE), name, color
INDEX (board_id)

-- card_labels (pivot)
card_id FK→cards (CASCADE), label_id FK→labels (CASCADE)
PK (card_id, label_id)

-- comments
id, card_id FK→cards (CASCADE), user_id FK→users (CASCADE),
body, timestamps, deleted_at
INDEX (card_id, created_at)

-- activity_log
id, board_id FK→boards (CASCADE), user_id FK→users (SET NULL),
action VARCHAR(40), subject_type, subject_id, properties JSONB
INDEX (board_id, created_at)
```

### Position ordering

**Integer positions + batch renumber** when gaps fall below 1. (Decimal/float is
the alternative but precision degrades.)

### Soft delete

- `SoftDeletes` on `boards`, `lists`, `cards`, `comments`.
- Not on `board_members`, `labels`, `card_labels`, `activity_log`.
- `users`: anonymize on deletion, don't soft-delete.
- Cascade: pivots + children. Top-level FKs to `users` are `RESTRICT`.

## REST API

All routes `/api/v1`. Authenticated routes use `auth:sanctum`.

### Auth

| Method | Path        | Body                                             | Returns         |
| ------ | ----------- | ------------------------------------------------ | --------------- |
| POST   | `/register` | `{name, email, password, password_confirmation}` | `{user, token}` |
| POST   | `/login`    | `{email, password}`                              | `{user, token}` |
| POST   | `/logout`   | (auth)                                           | 204             |
| GET    | `/me`       | (auth)                                           | `{user}`        |

### Boards

| Method | Path                   | Purpose                                  |
| ------ | ---------------------- | ---------------------------------------- |
| GET    | `/boards`              | List user's boards                       |
| POST   | `/boards`              | Create (creator = owner)                 |
| GET    | `/boards/{id}`         | Board with lists, cards, labels, members |
| PATCH  | `/boards/{id}`         | Update                                   |
| DELETE | `/boards/{id}`         | Soft delete                              |
| POST   | `/boards/{id}/restore` | Restore                                  |

### Board members

| Method | Path                            | Purpose         |
| ------ | ------------------------------- | --------------- |
| GET    | `/boards/{id}/members`          | List            |
| POST   | `/boards/{id}/members`          | Invite by email |
| PATCH  | `/boards/{id}/members/{userId}` | Change role     |
| DELETE | `/boards/{id}/members/{userId}` | Remove          |

### Lists

| Method | Path                 | Purpose          |
| ------ | -------------------- | ---------------- |
| POST   | `/boards/{id}/lists` | Create           |
| PATCH  | `/lists/{id}`        | Rename / reorder |
| DELETE | `/lists/{id}`        | Soft delete      |
| POST   | `/lists/{id}/move`   | `{position}`     |

### Cards

| Method | Path                | Purpose                                     |
| ------ | ------------------- | ------------------------------------------- |
| POST   | `/lists/{id}/cards` | Create                                      |
| GET    | `/cards/{id}`       | Detail with labels, assignees, comments     |
| PATCH  | `/cards/{id}`       | Update                                      |
| DELETE | `/cards/{id}`       | Soft delete                                 |
| POST   | `/cards/{id}/move`  | `{to_list_id, to_position}` — drag endpoint |

### Card relationships

| Method | Path                             | Purpose      |
| ------ | -------------------------------- | ------------ |
| POST   | `/cards/{id}/labels`             | Attach label |
| DELETE | `/cards/{id}/labels/{labelId}`   | Detach       |
| POST   | `/cards/{id}/assignees`          | Assign       |
| DELETE | `/cards/{id}/assignees/{userId}` | Unassign     |
| GET    | `/cards/{id}/comments`           | List         |
| POST   | `/cards/{id}/comments`           | Add          |
| DELETE | `/comments/{id}`                 | Delete       |

### Labels

| Method | Path                  | Purpose |
| ------ | --------------------- | ------- |
| GET    | `/boards/{id}/labels` | List    |
| POST   | `/boards/{id}/labels` | Create  |
| PATCH  | `/labels/{id}`        | Update  |
| DELETE | `/labels/{id}`        | Delete  |

### Activity

| Method | Path                    | Purpose       |
| ------ | ----------------------- | ------------- |
| GET    | `/boards/{id}/activity` | Paginated log |

## API resource shape

```jsonc
// GET /api/v1/boards/42
{
  "data": {
    "id": 42,
    "title": "Q4 Roadmap",
    "background": "blue",
    "owner": { "id": 1, "name": "Mushf" },
    "members": [{ "id": 1, "name": "Mushf", "role": "owner" }],
    "labels": [{ "id": 1, "name": "bug", "color": "red" }],
    "lists": [
      {
        "id": 100,
        "title": "Todo",
        "position": 1,
        "cards": [
          {
            "id": 500,
            "title": "Write API spec",
            "position": 1,
            "due_date": "2026-09-01",
            "assignees": [{ "id": 1, "name": "Mushf" }],
            "labels": [{ "id": 2, "name": "feature", "color": "green" }],
            "comment_count": 0,
          },
        ],
      },
    ],
  },
}
```

## Real-time events (Reverb)

All events broadcast on private channel `board.{id}`.

| Event                                                       | Payload                             |
| ----------------------------------------------------------- | ----------------------------------- |
| `CardMoved`                                                 | `{card, from_list_id, to_position}` |
| `CardCreated` / `CardUpdated` / `CardDeleted`               | `{card}` or `{card_id, list_id}`    |
| `ListCreated` / `ListUpdated` / `ListMoved` / `ListDeleted` | `{list}` or `{list_id}`             |
| `LabelCreated` / `LabelUpdated` / `LabelDeleted`            | `{label}`                           |
| `CommentCreated` / `CommentDeleted`                         | `{comment, card_id}`                |
| `MemberAdded` / `MemberRemoved` / `MemberRoleChanged`       | `{board_id, user, role?}`           |

Channel authorization:

```php
// routes/channels.php
Broadcast::channel('board.{boardId}', function (User $user, int $boardId) {
    return $user->boards()->where('boards.id', $boardId)->exists()
        || Board::find($boardId)?->owner_id === $user->id;
});
```

## Optimistic update (frontend)

```ts
// hooks/useMoveCard.ts
return useMutation({
  mutationFn: (v) =>
    api.post(`/cards/${v.cardId}/move`, {
      to_list_id: v.toListId,
      to_position: v.toPosition,
    }),
  onMutate: async (v) => {
    await queryClient.cancelQueries({ queryKey: ["board", boardId] });
    const snapshot = queryClient.getQueryData(["board", boardId]);
    queryClient.setQueryData(["board", boardId], (old) =>
      applyLocalMove(old, v),
    );
    return { snapshot };
  },
  onError: (_e, _v, ctx) =>
    ctx?.snapshot && queryClient.setQueryData(["board", boardId], ctx.snapshot),
  onSettled: () =>
    queryClient.invalidateQueries({ queryKey: ["board", boardId] }),
});
```

Echo subscription also calls `invalidateQueries` so peer moves re-render
automatically.

## Position renumbering (backend)

```php
// app/Services/CardMoveService.php
public function move(Card $card, int $toListId, int $toPosition): Card {
    return DB::transaction(function () use ($card, $toListId, $toPosition) {
        $oldListId = $card->list_id;
        $card->update(['list_id' => $toListId, 'position' => $toPosition]);

        Card::where('list_id', $oldListId)->where('position', '>', $card->getOriginal('position'))->decrement('position');
        Card::where('list_id', $toListId)->where('id', '!=', $card->id)->where('position', '>=', $toPosition)->increment('position');

        $this->renumberIfNeeded($toListId);
        event(new CardMoved($card->fresh()));
        return $card->fresh();
    });
}
```
