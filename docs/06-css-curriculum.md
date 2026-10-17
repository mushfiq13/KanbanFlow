# KanbanFlow — CSS Curriculum

> **Custom CSS is the default for this project.** Tailwind is **not** installed.
> There is no Tailwind budget because there is no Tailwind. This document is the
> source of truth for everything CSS in `web/`.

## Table of Contents

- [KanbanFlow — CSS Curriculum](#kanbanflow--css-curriculum)
  - [Table of Contents](#table-of-contents)
  - [The CSS Contract](#the-css-contract)
    - [C#/.NET analog](#cnet-analog)
  - [Topic Map](#topic-map)
  - [Layered Lessons](#layered-lessons)
    - [L1 — Cascade Layers](#l1--cascade-layers)
      - [What it is](#what-it-is)
      - [Why it exists](#why-it-exists)
      - [C#/.NET analog](#cnet-analog-1)
      - [When to reach for it](#when-to-reach-for-it)
      - [Common pitfalls](#common-pitfalls)
      - [How to drill it](#how-to-drill-it)
    - [L2 — Design Tokens + Custom Properties](#l2--design-tokens--custom-properties)
    - [L3 — Reset, Logical Properties, and the Box Model](#l3--reset-logical-properties-and-the-box-model)
    - [L4 — Modern Selectors: `:where()`, `:is()`, `:has()`, `:not()`](#l4--modern-selectors-where-is-has-not)
    - [L5 — CSS Nesting](#l5--css-nesting)
    - [L6 — Container Queries](#l6--container-queries)
      - [What it is](#what-it-is-1)
      - [Why it exists](#why-it-exists-1)
      - [C#/.NET analog](#cnet-analog-2)
      - [When to reach for it](#when-to-reach-for-it-1)
      - [Common pitfalls](#common-pitfalls-1)
    - [L7 — `color-mix()` and Relative Colors](#l7--color-mix-and-relative-colors)
    - [L8 — `@scope`](#l8--scope)
    - [L9 — `@starting-style` and the `prefers-reduced-motion` Contract](#l9--starting-style-and-the-prefers-reduced-motion-contract)
      - [What `@starting-style` is](#what-starting-style-is)
      - [The reduced-motion contract](#the-reduced-motion-contract)
      - [C#/.NET analog](#cnet-analog-3)
      - [Common pitfalls](#common-pitfalls-2)
    - [L10 — Fluid Type and Spacing with `clamp()`](#l10--fluid-type-and-spacing-with-clamp)
  - [Post-CSS Mastery Test](#post-css-mastery-test)
    - [Part 1 — Concept recall (10 questions, no notes, no Google)](#part-1--concept-recall-10-questions-no-notes-no-google)
    - [Part 2 — Snippet writing (open notes, 30 minutes)](#part-2--snippet-writing-open-notes-30-minutes)
    - [Part 3 — Build a feature (90 minutes, open notes, real repo)](#part-3--build-a-feature-90-minutes-open-notes-real-repo)
  - [Appendix — The `globals.css` Skeleton](#appendix--the-globalscss-skeleton)
    - [Token-addition protocol](#token-addition-protocol)

---

## The CSS Contract

Read this once. Internalize it. Then read the lessons.

1. **One `src/styles/globals.css`** at the project root, imported once in
   `src/main.tsx` before any component imports. No exceptions. No per-component
   global imports.
2. **Four cascade layers, in this order**:
   `@layer reset, base, components, utilities;`. Unlayered styles always win
   against layered styles — never write styles outside these four layers.
3. **Design tokens are CSS custom properties** declared inside
   `@layer base { :root { … } }`. The token names live in
   `docs/06-css-curriculum.md` §Appendix. New tokens require a `Learn` paragraph
   and a commit message explaining why.
4. **Component styles use CSS Modules** (`Foo.module.css`). Named exports in
   PascalCase: `import styles from './Button.module.css'`. Class names are
   camelCase: `.root`, `.icon`, `.label`.
5. **No `@apply`.** It ties you to PostCSS and defeats the custom-CSS default.
6. **Logical properties from day one.** `margin-inline`, `padding-block`,
   `border-inline-start`, `inset-block-end`. Never `margin-left` / `padding-top`
   / `border-right` in component code. Reset declarations and global styles may
   use physical properties once where unavoidable.
7. **One global `@media (prefers-reduced-motion: reduce)` block** at the bottom
   of `globals.css` that disables transitions and animations globally. Every
   Interaction Choreography respects this. If you need to preserve motion for
   accessibility (e.g. a focus outline that _must_ be visible), do it via state
   change, not animation.
8. **No inline styles** in JSX except for genuinely dynamic values (drag
   offsets, percent-based progress bars). Inline styles defeat the cascade, the
   contract, and the curriculum.
9. **No utility-class framework.** Tailwind is not installed. UnoCSS is not
   installed. Every class name is a class you wrote, in a file you own, in a
   module that scope-limits it.

### C#/.NET analog

The four-layer cascade is conceptually close to ASP.NET request middleware
ordering: `reset` is the exception handler, `base` is the auth middleware (runs
first, applies to everyone), `components` are your route handlers, `utilities`
are response compression (always wins, but you almost never reach for it). The
reason for the order is the same — **predictability of overrides**. If you break
the order, you ship a debugging week.

---

## Topic Map

There are 10 topics, grouped by when they appear in the Phase plan
(`docs/03-roadmap.md`).

| ID  | Topic                                      | Phase | Variant                                        |
| --- | ------------------------------------------ | ----- | ---------------------------------------------- |
| L1  | Cascade layers                             | 1     | **Deep Lesson**                                |
| L2  | Design tokens + custom properties          | 1     | Lesson                                         |
| L3  | Reset + logical properties                 | 1     | Lesson                                         |
| L4  | `:where()` / `:is()` / `:has()` / `:not()` | 2 / 4 | Lesson (`:has()` → **Deep Lesson** in Phase 4) |
| L5  | CSS nesting                                | 2     | Lesson                                         |
| L6  | Container queries                          | 3     | **Deep Lesson**                                |
| L7  | `color-mix()` + relative colors            | 3     | Lesson                                         |
| L8  | `@scope`                                   | 5     | Lesson                                         |
| L9  | `@starting-style` + reduced motion         | 4     | **Deep Lesson**                                |
| L10 | `clamp()` for fluid type / spacing         | 2     | Lesson                                         |

Each Phase ships at least one **CSS-only Monday** Task — a UI Task implemented
entirely with custom CSS, zero new utility classes (there are no utility classes
in the project, so the rule reduces to: at least one Task per Phase where the
CSS is the deliverable, not a side-effect).

---

## Layered Lessons

### L1 — Cascade Layers

**Deep Lesson (Phase 1, before any UI work).**

#### What it is

CSS cascade layers (`@layer`) are a 2022 browser-shipped feature that let you
**declare the order of author-origin styles independent of specificity**.
Without layers, the rule "last-declared wins at equal specificity" applies. With
layers, the rule becomes "last-declared _layer_ wins, full stop." Specificity
inside a layer is irrelevant to the cross-layer ordering.

```css
/* In globals.css — declare layer order ONCE, near the top */
@layer reset, base, components, utilities;

/* In Card.module.css */
@layer components {
  .title {
    color: red; /* high specificity? no */
  }
  .title.is-urgent {
    /* higher specificity? yes */
    color: blue;
  }
}
```

In the example above, even though `.title.is-urgent` has higher specificity than
`.title`, both are in `@layer components`. The order between them is the
standard specificity rule. But if a
`@layer utilities { .title { color: green; } }` existed elsewhere, the
`utilities` declaration would beat both regardless of specificity — because
`utilities` is declared last in the layer order at the top of `globals.css`.

#### Why it exists

Specificity wars are a tax. They lead to `!important` escalation, to "add
another class" debugging, to frameworks that simulate isolation. Cascade layers
solve the _root_ problem — not by hiding it (like scoped class names) but by
making the resolution order _explicit and author-declared_. Once you commit to a
four-layer contract, the question "why didn't my style apply?" has a small,
mechanical answer: "it's in a layer that comes before the one you expected."

#### C#/.NET analog

A .NET engineer already understands this. **Middleware ordering in
`Program.cs`**:

```csharp
app.UseExceptionHandler();     // reset: catches everything
app.UseAuthentication();       // base: applies to everyone
app.UseAuthorization();        // base: gates access
app.MapControllers();          // components: your routes
app.UseResponseCompression();  // utilities: last-mile
```

You would never put `UseResponseCompression()` _before_ `UseAuthentication()`
and expect anything to work. Cascade layers encode the same idea in CSS:
declaration order at the top of `globals.css` is the project's middleware
pipeline.

#### When to reach for it

- **Always, at the project level.** The first CSS you write in a new project
  should be `@layer reset, base, components, utilities;` at the top of
  `globals.css`. Not a card-by-card decision.
- **Inside a single CSS Module, rarely.** Use the same layer name
  (`@layer components { … }`) so your module is part of the project's
  `components` layer. Don't create new layers ad-hoc.
- **Never** for one-off overrides. If you're tempted to add a `@layer overrides`
  block to "win" against something in `components`, the answer is to fix the
  original — usually by reordering or by re-checking the cascade layers first.

#### Common pitfalls

- **Unlayered styles beat layered styles.** A bare `body { color: red; }` in a
  component file (no `@layer` wrapper) will win against every layered
  declaration. This is intentional in the spec — it's "the escape hatch" — but
  in this project it is a bug. The CSS-leaderboard lint
  (`web/scripts/css-leaderboard.sh` once it exists) should fail on any unlayered
  rule in component code. Until that script exists, code review must catch it.
- **You can't `@layer` around an `@import`.** Layer order is fixed by the first
  declaration; later `@layer` declarations just _add_ layers to the end of the
  existing order.
- **Nesting inside a layer does not create sub-layers.** A selector nested 5
  levels deep inside `@layer components` is still `components`-layer. Don't use
  nesting to fake priority.

#### How to drill it

Open DevTools → Elements panel → click any element → open the **Layers** tab
(Chrome 99+). You should see four layers in declared order. If you see fewer,
you have an unlayered rule somewhere. The "winning" declaration will be
highlighted in the panel — if the "winner" is in a different layer than you
expected, the layer order in `globals.css` is wrong, not the selector.

---

### L2 — Design Tokens + Custom Properties

**Lesson.**

Custom properties (a.k.a. CSS variables) are the project's only allowed way to
refer to colors, spacings, font sizes, durations, and easings. The token names
are listed in the Appendix below. To add a new token, add it under
`@layer base { :root { … } }` in `globals.css` and document it in this file's
Appendix with one sentence of "when to use it." Magic numbers in component CSS
are a code-review fail.

```css
/* ❌ Bad: magic number, brittle, no theming path */
.button {
  padding: 12px 24px;
}

/* ✅ Good: token, themeable, scannable */
.button {
  padding: var(--space-sm) var(--space-md);
}
```

The values for `clamp()` (see L10) and `color-mix()` (see L7) are _also_ tokens.
Compose tokens; don't fork them.

---

### L3 — Reset, Logical Properties, and the Box Model

**Lesson.**

The reset lives in `@layer reset` in `globals.css`. It is **minimal**:
`*, *::before, *::after { box-sizing: border-box; }`,
`* { margin: 0; padding: 0; }`, plus a sane `body` line-height and the
logical-property `padding-inline` instead of `padding-left/right`.

Logical properties are the _only_ way to express inset / margin / padding /
border in component code. The reason is RTL readiness: `margin-inline-start`
flips automatically when `[dir="rtl"]` is applied at the root; `margin-left`
does not. We are not building RTL today, but every choice we make today is a
choice we won't have to undo tomorrow.

```css
/* ❌ Bad: physical properties bake in direction */
.card {
  margin-left: var(--space-md);
}

/* ✅ Good: logical properties flip automatically */
.card {
  margin-inline-start: var(--space-md);
}
```

---

### L4 — Modern Selectors: `:where()`, `:is()`, `:has()`, `:not()`

**Lesson (Phase 2) for `:where()` / `:is()` / `:not()`.** **Deep Lesson
(Phase 4) for `:has()`** — drilled in the Skill Task before the card-modal UI
lands.

The mental model:

- `:where(a, b, c)` is exactly like `a, b, c` but its specificity is _zero_. Use
  it to make a long selector list not stomp on a component's local specificity.
- `:is(a, b, c)` is exactly like `a, b, c` but takes the _highest_ of their
  specificities. Use it to tighten a selector list into one selector without
  losing specificity.
- `:not(a, b, c)` matches anything that doesn't match. Specificity is the
  highest of the inner arguments.
- `:has(a)` is the _parent selector_. It matches an element that contains a
  descendant matching `a`. This was impossible before 2023; the conventional
  workaround was a class added by JS, which always desynced. **Phase 4 drills
  `:has()` because the card modal's "this card has overdue todos" red border
  depends on it.**

```css
/* The old way: two class additions, kept in sync by hand */
.card.has-overdue {
  border-color: red;
}

/* The new way: one rule, derived from the DOM */
.card:has(.todo.is-overdue) {
  border-color: red;
}
```

The deep lesson for `:has()` in Phase 4 covers: performance characteristics (it
invalidates more than a class toggle, so prefer it for "is X present" not "did X
animate"), the parent-flip nesting rules (`:has()` inside `:has()` is allowed
but expensive), and the fallback pattern (use a class toggled in JS for browsers
that don't support it — we target baseline 2024 so this is a non-issue, but the
convention is documented).

---

### L5 — CSS Nesting

**Lesson (Phase 2).**

Native CSS nesting (no preprocessor) is a 2023 browser-shipped feature. It is
allowed. The rule: nest **at most one level deep** in component files, and only
when the parent-child relationship is genuine (a button's hover state is a child
of the button, not a sibling). Three-level-deep nesting is a code-review fail
because the generated selector is no longer scannable.

```css
/* ✅ Good: one level deep, real relationship */
.button { … }
.button:hover { … }
.button:focus-visible { … }

/* ✅ Also good: parent wraps the family */
.button {
  …
  .icon { … }       /* becomes .button .icon */
  .label { … }      /* becomes .button .label */
}
```

---

### L6 — Container Queries

**Deep Lesson (Phase 3, before the boards list page).**

#### What it is

A `@container` query is like a media query, but instead of asking "how wide is
the viewport?", it asks "how wide is _this container_?" A card component inside
a 300px sidebar and a card component inside a 100% board column can have **the
same class name** but render differently, because each container is the source
of truth for its descendants.

```css
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

.card {
  display: grid;
  grid-template-columns: 1fr;
}

@container card (min-width: 400px) {
  .card {
    grid-template-columns: auto 1fr auto; /* cover | title | meta */
  }
}
```

The wrapper creates a containment context. The card inside responds to the
wrapper's width, not the viewport's. Reuse the same `<Card>` component in a
sidebar, in a board column, in a search result — it adapts to each.

#### Why it exists

Media queries are a viewport-era idea. In a world of sidebars, drawers, modals,
and grids, "the viewport" is the wrong unit. Container queries move the
responsive unit from the browser chrome to the _layout slot the component lives
in_. This is the single biggest "finally we can do this" CSS feature of the
2020s.

#### C#/.NET analog

`@container` is to a CSS component what **dependency injection** is to a C#
service. The component asks its environment "what are you?" instead of asking
the global "what's the world?" — and adapts.

#### When to reach for it

- **Any time a component is reused in more than one layout slot.** Card, list
  item, board tile, label chip, assignee avatar — all candidates.
- **Not for top-level page layouts.** The `<Board>` page itself responds to the
  viewport — that's a media query. The `<Card>` inside a list column responds to
  the column — that's a container query.

#### Common pitfalls

- **Forgetting `container-type`.** Without it, the element has no containment
  context and `@container` queries inside it are ignored. The shorthand
  `container: card / inline-size` works.
- **Container queries on flex/grid children** can cause layout-thrash if the
  child's own size determines the container's size. Avoid circular sizing.
- **Naming matters when nesting.** `container-name: card` is required if
  multiple containers are ancestors. Use the component's name.

---

### L7 — `color-mix()` and Relative Colors

**Lesson (Phase 3, before the labels feature).**

`color-mix(in oklab, var(--color-accent) 80%, white)` is the single most useful
color utility to land in CSS in years. It composes two colors in a
perceptually-uniform color space (`oklab` > `srgb` for this) and lets you derive
hover / active / disabled variants of a base token without forking the palette.

```css
:root {
  --color-accent: oklch(60% 0.18 250);
}
.button {
  background: var(--color-accent);
}
.button:hover {
  background: color-mix(in oklab, var(--color-accent) 85%, white);
}
.button:active {
  background: color-mix(in oklab, var(--color-accent) 70%, black);
}
```

The pattern: define a base token. Derive interactive states with `color-mix()`.
Never hand-pick a "hover color" — derive it.

---

### L8 — `@scope`

**Lesson (Phase 5, before the card detail modal).**

`@scope (.card) to (.card__body)` is a way to limit selector reach to a subtree.
It is the formal answer to "I want this style to apply to _this card_ and not
leak to _its parent or children_."

We will use it sparingly — most scoping is handled by CSS Modules. But for the
card detail modal, where the body content is user-provided HTML (descriptions,
comments), `@scope` is the defense against the user's `h1` styling the rest of
the app.

---

### L9 — `@starting-style` and the `prefers-reduced-motion` Contract

**Deep Lesson (Phase 4, before the card detail modal's enter animation).**

#### What `@starting-style` is

When an element transitions from `display: none` to `display: block` (or from
being absent to present, as in a dialog), the browser normally has no "before"
state to transition _from_. `@starting-style` provides one:

```css
.modal {
  opacity: 1;
  transform: translateY(0);
  transition:
    opacity 200ms var(--ease-out),
    transform 200ms var(--ease-out);
}

@starting-style {
  .modal {
    opacity: 0;
    transform: translateY(8px);
  }
}
```

Without `@starting-style`, the modal would just _appear_. With it, it fades in
from below.

#### The reduced-motion contract

The single global block in `globals.css` is the project-wide **enforcer** of the
`prefers-reduced-motion: reduce` media query:

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

This is intentionally aggressive. It kills **all** motion including decorative
fades, because the cost of ignoring a user's OS-level preference is higher than
the cost of losing a fade. Every Interaction Choreography in every Task MUST
respect this. The canonical pattern:

```
resting (effect, ms, easing) →
hover (effect, ms, easing) →
focus-visible (effect, ms, easing) →
active (effect, ms, easing)
prefers-reduced-motion: reduce → <what's kept, what's dropped>
```

The "what's kept" usually includes focus-visible outlines (they're assistive,
not decorative). The "what's dropped" usually includes transitions on color,
opacity, transform. Document both.

#### C#/.NET analog

The `prefers-reduced-motion` global block is the CSS equivalent of **a top-level
try/catch in `Program.cs` that swallows all decorative exceptions**. You still
have a happy path; you also have a guaranteed-safe fallback for users who said
"I don't want this."

#### Common pitfalls

- **Forgetting the `!important`.** Without it, a specific transition in
  component CSS will win. With it, the global enforcer wins. The intent is "the
  user said no motion; honor it." `!important` here is correct, not lazy.
- **Animations that _need_ to run for correctness** (e.g., a loading spinner
  that signals "we're working") should opt out with a local
  `@media (prefers-reduced-motion: no-preference)` wrapper. Don't
  blanket-disable a spinner that the user _needs_ to know is spinning.
- **`@starting-style` + `display: none`** is the only way to animate an element
  that starts hidden. If you use `visibility: hidden` + `opacity: 0`, the
  transition still works but `@starting-style` is unnecessary — the "before"
  state is just the resting state with opacity 0.

---

### L10 — Fluid Type and Spacing with `clamp()`

**Lesson (Phase 2, before the boards list page).**

`clamp(min, preferred, max)` lets a single declaration grow smoothly between a
minimum and a maximum. For type, the "preferred" is a `vw`-based value that
scales with the viewport but is bounded so it doesn't get unreadably large on
wide screens:

```css
:root {
  --font-size-display: clamp(2rem, 4vw + 1rem, 4rem);
  --space-section: clamp(2rem, 6vw, 6rem);
}
```

The pattern: pick a _minimum_ (smallest acceptable on mobile), a _fluid_
(`vw`-based), and a _maximum_ (largest acceptable on desktop). The browser
interpolates between them. No media query breakpoints needed for typography.

---

## Post-CSS Mastery Test

> **Three parts. After completing all 12 features, take this test in one
> sitting, no notes. Target: 30 minutes for Parts 1+2, 90 minutes for Part 3.**

### Part 1 — Concept recall (10 questions, no notes, no Google)

1. State the four cascade layers in declaration order.
2. What is the specificity of `:where(.a, .b, .c)`?
3. What is the specificity of `:is(.a, .b, .c)` if `.a` is a class, `.b` is a
   class, and `.c` is an ID?
4. What two properties must an element have to create a `@container` context?
5. What color space should `color-mix()` use for hover/active variants, and why?
6. What does `@starting-style` do, and what problem does it solve that
   `transition` alone cannot?
7. Name three modern selectors that became browser-native after 2020.
8. What is the project rule on `@apply`, and why?
9. What does the global `prefers-reduced-motion: reduce` block do, and what is
   the one thing it should _not_ blanket-disable?
10. What is the difference between a media query and a container query, in one
    sentence?

### Part 2 — Snippet writing (open notes, 30 minutes)

For each of the following, write the CSS that satisfies the brief. Use only the
four cascade layers, only the project tokens, and only custom properties. No
`@apply`, no inline styles, no utility classes.

1. A `<Button>` component with three states (resting, hover, focus-visible,
   active) using `color-mix()` for the state variants. The button's resting
   background must be `--color-accent`. The hover must be 15% lighter. The
   active must be 20% darker. The focus ring is 2px solid `--color-focus`,
   `outline-offset: 4px`, no transition (instant, for a11y). The reduced-motion
   contract removes the color transitions on hover and active but keeps the
   focus ring.
2. A `<Card>` component that re-arranges its layout based on its container's
   width: stacked (1 column) under 400px, horizontal (3 columns: cover / title /
   meta) at 400px and above. Use a container query. Use a CSS Module. The card's
   wrapper declares the container.
3. A `<Modal>` component that fades in from below when opened, using
   `@starting-style` and the project's `--ease-out` token. Includes the
   `prefers-reduced-motion` fallback in a comment block.

### Part 3 — Build a feature (90 minutes, open notes, real repo)

Pick the hardest UI Task from the Phase you just finished. Re-implement its CSS
from scratch using only:

- The four cascade layers
- The project tokens
- CSS Modules
- Container queries where the layout demands them
- `@starting-style` if the element enters the DOM dynamically
- `color-mix()` for state variants
- Logical properties throughout

Then open DevTools → Layers panel → confirm all your styles are in the expected
layer. Then toggle `prefers-reduced-motion: reduce` in the rendering tab and
confirm the choreography still works (assistive, not decorative).

If you can do all three parts cleanly, you've mastered the curriculum.

---

## Appendix — The `globals.css` Skeleton

This is the canonical `src/styles/globals.css`. The T-1.4 worked example (in
`docs/05-task-and-skill-task-templates.md`) adapts it for the Vite/React
scaffold; the structure is unchanged.

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
    padding-inline: 0;
  }
}

/* Design tokens */
@layer base {
  :root {
    /* Color */
    --color-bg: #fafaf9;
    --color-surface: #ffffff;
    --color-text: #1a1a1a;
    --color-muted: #6b6b6b;
    --color-accent: oklch(60% 0.18 250);
    --color-focus: oklch(70% 0.2 250);
    --color-danger: #dc2626;
    --color-success: #16a34a;
    --color-border: #e7e5e4;

    /* Typography */
    --font-sans:
      ui-sans-serif, system-ui, -apple-system, "Segoe UI", sans-serif;
    --font-display: var(--font-sans);
    --font-mono: ui-monospace, "SF Mono", Menlo, monospace;
    --font-size-sm: 0.875rem;
    --font-size-base: 1rem;
    --font-size-lg: 1.125rem;
    --font-size-display: clamp(2rem, 4vw + 1rem, 4rem);

    /* Spacing */
    --space-xs: 0.25rem;
    --space-sm: 0.5rem;
    --space-md: 1rem;
    --space-lg: 1.5rem;
    --space-xl: 2rem;
    --space-section: clamp(2rem, 6vw, 6rem);

    /* Radius */
    --radius-sm: 4px;
    --radius-md: 8px;
    --radius-lg: 12px;

    /* Elevation */
    --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.06);
    --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.08);
    --shadow-lg: 0 12px 32px rgba(0, 0, 0, 0.12);

    /* Motion */
    --duration-fast: 80ms;
    --duration-base: 200ms;
    --duration-slow: 360ms;
    --ease-out: cubic-bezier(0.2, 0.8, 0.2, 1);
    --ease-in: cubic-bezier(0.4, 0, 1, 1);
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
  /* populated only when a genuinely hand-rolled utility beats a component */
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

### Token-addition protocol

1. Propose the new token in a PR description with one sentence: "When would a
   component reach for this?"
2. Add the declaration under `@layer base { :root { … } }` in `globals.css`.
3. Add the token to this Appendix with the same one-sentence rationale.
4. Reference the token in at least one component before merging. Unreferenced
   tokens are dead code; remove them.
