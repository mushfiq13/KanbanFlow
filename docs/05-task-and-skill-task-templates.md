# KanbanFlow — Task Templates and Worked Examples

This is a **reference / how-to-use document**, not part of the curriculum
content itself. Read it before writing or running any Task. After you've
internalized the template, you should not need to re-read this file.

---

## Table of Contents

1. [The Hierarchy](#the-hierarchy)
2. [Task Template](#task-template)
3. [Skill Task Template](#skill-task-template)
4. [Worked Example: T-1.2 (Backend Scaffold)](#worked-example-t-12-backend-scaffold)
5. [Worked Example: T-1.4 (Frontend Scaffold)](#worked-example-t-14-frontend-scaffold)
6. [CSS Strategy Lock-In](#css-strategy-lock-in)
7. [Common Mistakes](#common-mistakes)

---

## The Hierarchy

```
Phase (1-week milestone)
  └─ Epic (EPIC-N)
       └─ Task (one per user story; ~84 total)
            ├─ Skill Task (CSS focus; ~16 across all Phases)
            └─ Stretch (per-Phase, only when too large)
```

- **Phase** = a 1-week milestone from the 8-week plan.
- **Epic** = `EPIC-N` is the unit label. Keep the existing EPIC IDs.
- **Task** = one existing user story. A vertical slice that exercises one
  Laravel / Next.js / CSS concept.
- **Skill Task** = CSS-focused micro-Task that drills one modern CSS topic
  _before_ the UI Task that uses it. Has a **Lesson** or **Deep Lesson** block.
- **Stretch** = per-Phase optional sub-steps. One per Phase at most.

---

## Task Template

Every Task has these fields, in this order. Don't skip fields. Don't reorder.

```markdown
### T-EPIC.STORY (Task title)

**What to do**: One sentence. Verb-first. Names the deliverable, not the
journey. **Files to touch**: Absolute or repo-relative paths. Grouped if many.
**Exit check**: A command, an assertion, or an observable state. The "definition
of done" for this Task. **CSS to ship**: _(UI Tasks only)_ One sentence
describing what the user **sees**, not what was drilled. Omit for non-UI Tasks.
**Learn**: A real paragraph (3+ sentences) explaining the conceptual "why" —
name the specific Laravel / Next.js / CSS concept, the C#/.NET analog if there
is one, and one pitfall. **Interaction Choreography**: _(UI Tasks only)_ A prose
state diagram of motion. Format:
`state (effect, timing, easing) → state (effect, timing, easing) → …`. Include
`prefers-reduced-motion` behavior.
```

### When to use which fields

| Task type              | What | Files | Exit check | CSS to ship | Learn | Choreography                                              |
| ---------------------- | ---- | ----- | ---------- | ----------- | ----- | --------------------------------------------------------- |
| Backend (Laravel)      | ✅   | ✅    | ✅         | —           | ✅    | —                                                         |
| Frontend logic (no UI) | ✅   | ✅    | ✅         | —           | ✅    | —                                                         |
| UI Task                | ✅   | ✅    | ✅         | ✅          | ✅    | ✅                                                        |
| Test-only              | ✅   | ✅    | ✅         | —           | ✅    | —                                                         |
| Skill Task             | ✅   | ✅    | ✅         | (optional)  | ✅    | — (Lesson / Deep Lesson instead, see Skill Task Template) |

---

## Skill Task Template

A Skill Task drills one CSS topic _in isolation_, in product code, before the UI
Task that uses it. Most Phases have 1–3 Skill Tasks. The Phases that need 2
(e.g., Phase 2 needs cascade layers + reset+tokens+logical-properties) get two;
the Phases that need only one (e.g., Phase 3 needs flexbox alone) get one.

```markdown
### ST-CSS-PHASE.N (Skill Task title)

**What to do**: One sentence. Verb-first. Names the deliverable. **Files to
touch**: Paths, usually in `web/` only. **Exit check**: A command or DevTools
assertion. (e.g., "open DevTools → Layers panel → see four layers in declared
order.") **CSS to ship**: _(optional)_ One sentence describing what the
resulting CSS looks like. **Learn**: A real paragraph (3+ sentences) on the
specific CSS concept being drilled. **Consumed by**: Bullet list of UI Task IDs
that cite this Skill Task. **Required** — Skill Tasks without inbound citations
must be removed or paired. **Lesson** OR **Deep Lesson**: A concept explanation.
**Lesson** = 3–5 sentences (most topics). **Deep Lesson** = 500–1000 words (only
for: cascade layers, `:has()`, container queries, `@starting-style`). Includes:
what it is, why it exists, a C#/.NET analog, when to reach for it, common
pitfalls.
```

### When to use Deep Lesson vs Lesson

| Topic                                                       | Variant         |
| ----------------------------------------------------------- | --------------- |
| Cascade layers                                              | **Deep Lesson** |
| `:has()`                                                    | **Deep Lesson** |
| Container queries                                           | **Deep Lesson** |
| `@starting-style`                                           | **Deep Lesson** |
| Everything else CSS                                         | Lesson          |
| Non-CSS (Laravel / Next.js concepts inside Skills are rare) | Lesson          |

---

## Worked Example: T-1.2 (Backend Scaffold)

Internalize this. Every other Task follows the same shape.

### T-1.2 Scaffold the Laravel 12 backend at `api/`

**What to do**: Run `composer create-project` at the repo root to materialize a
Laravel 12 project in `api/`, then install the API surface with
`php artisan install:api`.

**Files to touch**:

- New: `api/` (entire Laravel project)
- New: `api/routes/api.php` (created by `install:api`)
- Modified: `composer.json` (auto-modified by composer)

**Exit check**:

```bash
cd api && php artisan --version
# Expected: Laravel Framework 12.x.x
cd api && cat routes/api.php | head -20
# Expected: a routes file that imports something — proves install:api ran
```

**Learn**: `composer create-project laravel/laravel:^12 api` does three things
at once: (a) it pulls the Laravel skeleton (the same skeleton every Laravel app
starts with), (b) it runs `composer install` to download dependencies into
`vendor/`, and (c) it copies a `.env.example` to `.env` and generates an
`APP_KEY`. The `:^12` is a Composer version constraint — it pins to the major
version 12 with any minor/patch, so future `composer update` calls won't
accidentally bump you off Laravel 12. C#/.NET analog: this is like
`dotnet new webapi -n Api` plus `dotnet restore` plus
`dotnet user-secrets init`, all chained in one command. The
`php artisan install:api` step adds the API routes file and Sanctum — those are
NOT in the default skeleton, you opt in. Pitfall: do NOT pass `--prefer-dist` if
you're on a slow network — `create-project` will fail silently and leave you
with a half-cloned `api/`. If `php artisan install:api` fails, check that `php`
has the `mbstring` and `xml` extensions.

**CSS to ship**: _n/a — backend Task._

---

## Worked Example: T-1.4 (Frontend Scaffold)

### T-1.4 Scaffold the Next.js 15 frontend at `web/` with custom-CSS strategy locked in

**What to do**: Run `pnpm create next-app` at the repo root to materialize a
Next.js 15 project in `web/` with TypeScript, App Router, ESLint, Tailwind (kept
but demoted), and the custom-CSS strategy pre-installed in `app/globals.css`.

**Files to touch**:

- New: `web/` (entire Next.js project)
- New: `web/app/globals.css` (rewritten with `@layer` block, design tokens,
  logical properties)
- New: `web/scripts/css-leaderboard.sh` (Tailwind budget lint)
- Modified: `web/app/page.tsx` (light replace if needed)
- Modified: `web/README.md` (add a one-paragraph note: "Tailwind is installed
  but demoted; see `docs/01-stack.md` §Styling")

**Exit check**:

```bash
cd web && pnpm dev
# Expected: dev server on http://localhost:3000, returns 200
cd web && bash scripts/css-leaderboard.sh
# Expected: exit 0 (no Tailwind usage yet, budget has headroom)
```

In a second browser tab, verify the default Next.js page loads. Then open
DevTools → Elements → confirm the body's computed `font-family` references
`--font-sans` from the `app/globals.css` `@layer base` block.

**CSS to ship**: A two-tone landing page (light gray background `--color-bg`,
near-black text `--color-text`, a centered "KanbanFlow" wordmark in
`--font-display`) that does not flash on first paint — the design tokens must be
inlined in `app/globals.css` so they hit the first paint.

**Learn**: `pnpm create next-app@latest web` is the official scaffolder. With
`--ts --app --eslint --src-dir --tailwind --import-alias "@/*"`, it produces a
Next.js 15 App-Router project skeleton. App Router (vs Pages Router) means the
project uses `app/page.tsx`, `app/layout.tsx`, server components by default, and
`'use client'` boundaries to opt in to interactivity. The `--tailwind` flag
installs Tailwind (we keep it, but we'll only use it for trivial helpers — see
below). C#/.NET analog: App Router + Server Components ≈ ASP.NET MVC's Razor
Pages; each `page.tsx` is a Razor view, and `'use client'` marks a Razor
component as having client-side state. Pitfall: Next.js 15 ships with
**Turbopack** as the default dev bundler, which is fast but aggressive about
caching — if `globals.css` looks unchanged when you edit, run `rm -rf .next` and
restart.

**Interaction Choreography** (for the landing-page wordmark):

- **resting** (opacity 1, letter-spacing normal, no shadow, 0ms)
- **hover** (opacity 0.85, letter-spacing +0.02em via CSS
  `transition: letter-spacing 200ms ease-out, opacity 200ms ease-out`)
- **focus-visible** (same as hover +
  `outline: 2px solid var(--color-accent); outline-offset: 4px`, transition
  80ms)
- **active / mousedown** (opacity 0.7, 60ms ease-in)
- `prefers-reduced-motion: reduce` → opacity changes and outline are kept
  (assistive, not decorative); letter-spacing is removed.

---

## CSS Strategy Lock-In

`api/` is plain Laravel — no CSS work there beyond what's in `resources/css`
(we'll set that up in T-2.x when the Auth UI arrives).

`web/` ships with Tailwind installed but with a strict policy enforced by
`web/scripts/css-leaderboard.sh`. The policy:

1. **Custom CSS is the default** for: layout, theming, animations, state
   changes, design tokens, component styling, anything you'd write a Learn
   paragraph about.
2. **Tailwind is allowed** for: trivial spacing (`mt-2`, `px-4`), color helpers
   (`text-gray-500`), pre-built accessible primitives (`sr-only`).
3. **CSS-only Monday**: at least one Task per Phase must be implemented with
   zero utility classes.
4. **The leaderboard lint** checks: counts of `className` and `class=`
   attributes in `web/components/**` and `web/app/**` against a per-Phase
   budget. Over budget → exit 1.

### Per-Phase Tailwind Budget (initial values; revised each Phase)

| Phase | Max utility-class usages across `web/components/**` and `web/app/**` |
| ----- | -------------------------------------------------------------------- |
| 1     | 0 (`web/` doesn't exist)                                             |
| 2     | 8 (mostly layout primitives)                                         |
| 3     | 16 (boards page is mostly custom)                                    |
| 4     | 24                                                                   |
| 5     | 32 (drag UI is custom; Tailwind for spacing only)                    |
| 6     | 40                                                                   |
| 7     | 48                                                                   |
| 8     | 56                                                                   |

These are _starting_ values. The lint script lives at
`web/scripts/css-leaderboard.sh` (T-1.4 deliverable) and reads the budget from
`web/.css-leaderboard-budget.json` (also T-1.4). Each Phase's Tasks should aim
for _less than budget_ — the closer to 0, the more you learned.

### What goes in `app/globals.css` (T-1.4 deliverable)

```css
@layer reset, base, components, utilities;

/* Reset */
@layer reset {
  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }
  * {
    margin: 0;
    padding: 0;
  }
  body {
    min-height: 100vh;
    line-height: 1.5;
  }
  /* Logical properties for RTL readiness */
  body {
    padding-inline: 0;
  }
}

/* Design tokens */
@layer base {
  :root {
    --color-bg: #fafaf9;
    --color-surface: #ffffff;
    --color-text: #1a1a1a;
    --color-muted: #6b6b6b;
    --color-accent: #2563eb;
    --color-danger: #dc2626;

    --font-sans:
      ui-sans-serif, system-ui, -apple-system, "Segoe UI", sans-serif;
    --font-display: var(--font-sans);

    --space-xs: 0.25rem;
    --space-sm: 0.5rem;
    --space-md: 1rem;
    --space-lg: 1.5rem;
    --space-xl: 2rem;

    --radius-sm: 4px;
    --radius-md: 8px;
    --radius-lg: 12px;

    --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.06);
    --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.08);

    --duration-fast: 80ms;
    --duration-base: 200ms;
    --duration-slow: 360ms;
    --ease-out: cubic-bezier(0.2, 0.8, 0.2, 1);
  }

  body {
    background: var(--color-bg);
    color: var(--color-text);
    font-family: var(--font-sans);
  }
}

@layer components {
  /* populated in Phase 2+ */
}
@layer utilities {
  /* populated only when a Tailwind output is genuinely hand-rolled */
}

/* Honor reduced motion globally */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

This file is the **CSS foundation** of the project. Don't restructure the layers
without a Learn paragraph and a commit message explaining why.

---

## Common Mistakes

These are the mistakes we see over and over. Read this section once before your
first Task.

### Tailwind seepage

**Symptom**: A UI Task's CSS-to-ship field describes a layout, and the resulting
code uses `className="flex items-center gap-4"` for the structure.

**Fix**: Drop the utilities. Write the equivalent CSS in a `Foo.module.css`
file. Reference the Skill Task for the concept (flexbox / grid / custom
property).

### CSS in `Learn` paragraphs

**Symptom**: A Learn paragraph says "this Task is about flexbox."

**Fix**: The Learn paragraph is for _Laravel / Next.js / architectural
concepts_. CSS concepts live in the Skill Task's Lesson / Deep Lesson. UI Tasks'
Learn paragraphs may mention CSS, but only as a side-effect — e.g., "this Task
uses `display: flex` from ST-CSS-3.1."

### Exit checks that don't actually check

**Symptom**: `cd web && pnpm dev` and then "look at it." The Exit check is "the
dev server is running."

**Fix**: Exit checks must be **machine-verifiable** OR **observable through a
single command**. Examples that work: `pnpm test`,
`php artisan test --filter=FooTest`, `curl -I http://localhost:3000`. Examples
that don't: "the page looks good," "the animation feels right."

### Interaction Choreography that's a one-liner

**Symptom**: "Choreography: fades in over 200ms."

**Fix**: A Choreography is a state diagram. Format:
`resting (X, Yms, easing) → hover (X, Yms, easing) → focus (X, Yms, easing) → active (X, Yms, easing)`.
Always include `prefers-reduced-motion` behavior at the end. See T-1.4 above for
the canonical example.

### Skill Tasks without inbound references

**Symptom**: A Skill Task is written but no UI Task cites it in its CSS-to-ship
or Learn fields.

**Fix**: Pair every Skill Task with a UI Task that uses the concept, in the same
Phase. If no such UI Task exists, the Skill Task is over-scoped and should be
removed.

### Stretch Tasks per-Task instead of per-Phase

**Symptom**: A UI Task ends with "Stretch: do X."

**Fix**: Stretch is **per-Phase**, not per-Task. If you find yourself wanting to
add a Stretch to a Task, either the Task is too large (split it) or the Stretch
isn't needed (drop it).

---

## What comes next

> **Forward-looking — Step 6 reference.** This block previews the order in which
> Tasks T-1.1, T-1.2, and T-1.4 will be _executed_ (Step 6 of the plan). They do
> **not** exist as T-IDs in `docs/04-user-stories.md` yet — that file uses
> `US-N.M` IDs and will be renamed to `T-N.M` during Step 4 (mass epic rewrite).
> Until Step 4 lands, treat any `T-N.M` references in this doc as plan-internal
> shorthand.

After T-1.2 and T-1.4 land:

1. **Pre-Phase 1 5-bullet reading summary** → write
   `docs/journal/phase-1-summary.md` from any "Read first" resources in
   `docs/03-roadmap.md` Phase 1. (Phase 1 is bootstrap; the summary may be
   brief.)
2. **T-1.1** → verify the toolchain: `php --version`, `composer --version`,
   `node --version`, `pnpm --version`, `psql --version`, `docker --version`.
   Each ≥ minimum.
3. **T-1.3** and **T-1.5** are EPIC-1's two remaining Tasks; they're in
   `docs/04-user-stories.md` (still in original voice; they'll be rewritten in
   Step 4).
