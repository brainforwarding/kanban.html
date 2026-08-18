# board — design brief

A personal kanban for one developer who runs AI coding agents (Claude Code, Codex) in the terminal.

## Hard requirements (from the client)

1. **Runs in the browser from local files.** Double-click `index.html`. Vanilla HTML/CSS/JS. No build step, no framework, no network calls, no CDN.
2. **Storage is `localStorage` only.** No database, no sync, no accounts. Single user, single machine.
3. **Drag and drop** between stages, and reorder within a stage.
4. **Session field.** Every card can hold the resume command its agent printed, e.g.
   `claude --resume 2d2bb76b-e6df-46c5-b742-8eab8c3c7303`. It is a plain text field with a
   copy control, so the client can paste it into a terminal and continue that session.
5. **Projects.** Managed in their own section (add / rename / recolor / delete). When creating or
   editing a task, the project is picked from that list.
6. **All the important kanban functions**: create, edit, delete, reorder, move, rename stages,
   add/remove stages, filter, search, undo, backup.
7. **Weekly progress report.** A button generates a **Markdown file** listing every card that was
   **created** that week or **moved to another stage** that week, showing **initial state → end state**,
   **title only** (no description). Clean output.
   - Weeks run **Monday 00:00 → Sunday 23:59**, in **America/Santiago** (Chile).
   - The report is delivered Monday morning for the week that just ended.
   - Dates are stamped automatically, but the client must be able to **override the date** when
     registering work actually done in a different week.
   - **Every week is listed** and any past week's summary can be generated at any time,
     **full or partial** (a subset of the week's entries).
8. **Aesthetic**: "really cool and clean with awesome animations, but minimalistic, no distractions,
   no heavy text explanations either. great design is very intuitive."

## Subject and voice

The defining artifact of this board is not the card — it is the **resume command**. It comes from a
terminal, it is machine-generated, and it is the thing that gets the client back into flight.

Type direction follows from that: **the machine speaks in monospace** (stage names, counts, session
strings, dates, week ranges), **the human speaks in Avenir Next** (task titles, notes). Nothing else
in the UI is allowed to be monospace, so the mono voice always means "this came from the system."

Palette: graphite with a violet cast (`#14131A` → `#2E2C3A`), one accent — **amber `#FFB454`**, the
color of a terminal cursor. Project colors are the only other chroma on screen. Deliberately *not*
near-black + acid green (terminal cliché), *not* cream + serif + terracotta.

## Direction A — "Workbench" (current build)

Board is the whole screen. The report is a modal you summon.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ ▍board   ● All  ● crm 4  ● webapp 2  ● docs  +      [Search]  ⧉  ＋  ⋯   │  52px rail
├─────────────────────────────────────────────────────────────────────────────┤
│ INBOX      3        DOING     2        WAITING   1        DONE      8   [+] │  mono caps
│ ┌─────────────────┐ ┌─────────────────┐ ┌────────────────┐ ┌──────────────┐ │
│ │▌Fix webhook retr│ │▌Refactor auth   │ │▌PR #412 review │ │▌Ship v2      │ │
│ │ crm           ● │ │ crm             │ │ webapp      │ │ docs         │ │
│ │ ▸ claude --resu…│ │ ▸ codex resume …│ │                │ │              │ │
│ └─────────────────┘ └─────────────────┘ └────────────────┘ └──────────────┘ │
│ ┌─────────────────┐                                                          │
│ │▌Write changelog │   ← 2px left edge = project color                       │
│ └─────────────────┘                                                          │
└─────────────────────────────────────────────────────────────────────────────┘
```

Card anatomy — the session line is the signature element:

```
┌──────────────────────────────────┐
│▌ Fix webhook retries             │   title, Avenir 500/14
│  Retry with backoff, cap at 5    │   notes, 2-line clamp, muted
│  crm                          ●  │   project (in project color) · flag
│ ┌──────────────────────────────┐ │
│ │ ▸ claude --resume 2d2bb76b…⧉ │ │   mono 11, amber caret, click = copy
│ └──────────────────────────────┘ │
└──────────────────────────────────┘
      ↓ click to copy
┌──────────────────────────────────┐
│ │ ▸ copied to clipboard      ✓ │ │   amber sweep, reverts after 1.2s
└──────────────────────────────────┘
```

Report modal:

```
┌── ‹  MON 3 – SUN 9 MAR 2026  ›            [ this week ▾ ]   × ──┐
│    9 cards · 3 created · 4 finished                              │
├──────────────────────────────────────────────────────────────────┤
│ ☑ crm                                                            │
│   ☑ Fix webhook retries          INBOX → DONE          [3 Mar]   │
│   ☑ Refactor auth                NEW   → DOING         [5 Mar]   │
│ ☑ webapp                                                      │
│   ☐ PR #412 review               DOING → WAITING       [6 Mar]   │
├──────────────────────────────────────────────────────────────────┤
│ Select all                          Copy markdown   Download .md │
└──────────────────────────────────────────────────────────────────┘
```

## Direction B — "Week ledger"

Board on the left, a permanent narrow **week strip** on the right that logs every move as it happens
(today's entries at top). The report is not a modal you summon — it is always visible, and the
button just exports what you can already see.

```
┌──────────────────────────────────────────────┬──────────────────┐
│ INBOX     DOING     WAITING    DONE          │ WEEK 3–9 MAR  ⤓ │
│ ┌───┐     ┌───┐     ┌───┐      ┌───┐         │ ─────────────── │
│ │   │     │   │     │   │      │   │         │ TUE 4           │
│ └───┘     └───┘     └───┘      └───┘         │  Fix webhook    │
│ ┌───┐               ┌───┐                    │  inbox → doing  │
│ │   │               │   │                    │ MON 3           │
│ └───┘               └───┘                    │  Refactor auth  │
│                                              │  new → inbox    │
└──────────────────────────────────────────────┴──────────────────┘
```

Trade-off: the week is never out of sight, which suits someone who must report weekly — but it costs
~260px of board width permanently and adds a second scrolling surface. Against "no distractions."

## Direction C — "Command surface"

No rail. The board is the only chrome; everything (new task, project, report, week, theme) happens in
a ⌘K palette. Maximum minimalism, highest learning cost — bad fit for "great design is very intuitive"
unless the palette is discoverable from an empty state.

## Data model

```js
{
  v: 2,
  theme: 'dark' | 'light',
  asOf: null | '2026-03-05',        // date override for logging; null = today (Chile)
  columns: [{ id, name }],           // ordered = stage order
  projects: [{ id, name, color }],
  tasks:   [{ id, title, notes, projectId, session, flag, columnId, order, createdAt, updatedAt,
              archivedAt?, archivedFrom? }],  // set only once archived
  events:  [{ id, taskId, title, type: 'created'|'moved', from, to, at, day }],
  filter: null | projectId
}
```

`events` is the report's source of truth and is append-only.

- `from` / `to` store **stage names as strings**, snapshotted at the time of the event, so renaming
  or deleting a stage later cannot corrupt history.
- `title` is likewise snapshotted, but the report prefers the task's *current* title when the task
  still exists (so a fixed typo shows up in the report).
- `at` is epoch ms, used only for ordering within a day.
- `day` is the **`YYYY-MM-DD` calendar date in America/Santiago** — this, not `at`, is what the
  report groups by. That is what makes the date override work: overriding rewrites `day`, never `at`.

### Archive

Done is a buffer, not a graveyard and not a shredder. Finished cards get **archived**: they keep
their row in `tasks` and only gain `archivedAt` (epoch ms) and `archivedFrom` (the stage name,
snapshotted, because stages get renamed and deleted). Anything with `archivedAt` is filtered out of
the board, the stage counts and the project counts, and appears in the Archive panel instead.

**The board can only archive; only the archive can delete.** That is the whole safety property:
nothing on the board is one click from gone, and the single irreversible action lives in one place,
behind a two-step confirm (Delete → "Sure?", disarming after ~3s). No modal — a confirm dialog for a
one-person board is ceremony.

Neither archiving nor deleting can damage the weekly report, for different reasons. An archived task
still exists, so the report resolves its live title as usual. A deleted one is gone from `tasks`, so
the report falls back to the title snapshotted on its events and flags the row `deleted` — the same
path a deleted card always took. Clearing the board never costs you a week's history.

Adding and removing **stages** is deliberately not on this path: it is a once-a-year action, so it
carries no standing chrome. "Add stage" lives in the ⋯ menu, with a hairline `+` in the header row
that only appears while the pointer is on the board.

### Week math

`America/Santiago` is UTC−3 / UTC−4 with DST, so week boundaries can never be computed from raw
epoch arithmetic. Every date decision goes through the calendar date:

```js
ymd(ts)        // Intl.DateTimeFormat('en-CA', {timeZone:'America/Santiago'}) → "2026-03-05"
mondayOf(ymd)  // shift the *calendar* date back to Monday (ISO weekday), no epoch math
weekOf(ymd)    // { monday, sunday, label: "3–9 Mar 2026" }
```

### Report aggregation

For the selected week, walk that week's events in `(day, at)` order and fold them per task:

- first event's `from` → the entry's **initial state** (`New` if the first event is `created`)
- last event's `to` → the entry's **end state**
- a task created *and* moved in the same week shows `NEW → <final stage>`
- a task moved in an earlier week and again this week starts from the stage it was in *this* week
- deleted tasks still appear — the work happened
- entries group by project; unassigned tasks group under "No project"

Markdown output:

```markdown
# Progress — 3–9 Mar 2026

9 cards · 3 created · 4 finished

## crm
- Fix webhook retries — Inbox → Done
- Refactor auth — New → Doing

## webapp
- PR #412 review — Doing → Waiting

## No project
- Buy domain — New → Done
```

## Motion spec

| Moment | Behaviour |
|---|---|
| Card lift | pointer drag, 5px threshold, ghost scales 1.03 + rotates 0.6°, shadow deepens |
| Neighbours | FLIP: every card animates from its old rect to its new one, 260ms, `cubic-bezier(.2,.8,.25,1)` |
| Drop | ghost flies to the placeholder rect (190ms), then vanishes — no snap |
| Column hover | drop target column tints its background |
| Copy | amber sweep across the chip, label swaps to `copied to clipboard`, reverts after 1.2s |
| Card add | fade + rise 9px, scale .985 → 1 |
| Filter / search | FLIP again — cards visibly travel to their new positions instead of blinking |
| Modal | scale .985 + 10px rise, backdrop blur |
| Cursor | 6×13px amber block, 1.15s step-blink, in the wordmark |
| Reduced motion | all durations collapse to ~0 |

## Open questions for review

1. Is the report a **modal** (A), a **permanent panel** (B), or something else?
2. Where does the **date override** live so it is discoverable but never fires by accident?
3. Is the quick composer (type title inline in a column, Enter to add) enough, or should every
   creation open the full editor?
4. What is the **empty state** — the client explicitly does not want explanatory text.
5. Is one accent + project colors enough chroma, or does the board need stage-level color?
