# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

Open `index.html` directly in a browser — no server or build step required.

## Architecture

Everything lives in `index.html`. `ptg_convention_classes.md` is the original source data used to populate the embedded JavaScript (it is not loaded at runtime).

The file has three logical sections:

1. **CSS** (`<style>` block, lines ~8–290) — all styling, including CSS custom properties, category/level color systems, and responsive layout
2. **HTML** (lines ~292–353) — static shell; dynamic content areas are empty `<div>` placeholders populated by JavaScript
3. **JavaScript** (`<script>` block, lines ~354–end) — data, rendering logic, filters, and localStorage persistence

## Key Data Structures

### `schedule[]`
Array of class objects. Each entry:
```js
{ day, time, title, presenter, room, category, multi?, note? }
```
- `multi: true` marks classes spanning two periods (time shows full range, e.g. `"8:00–9:30 to 10:30–12:00"`)
- `note` is optional and renders with a ⚠️ warning

### `classInfo{}`
Maps class title → `{ level, desc }`. Sourced from ptgconvention.com. Keys must exactly match `title` values in `schedule[]`. Not every class has an entry.

## CSS Conventions

**Category border/label classes:** `cat-action`, `cat-business`, `cat-caut`, `cat-design`, `cat-etd`, `cat-forums`, `cat-health`, `cat-players`, `cat-rebuilding`, `cat-rpt`, `cat-service`, `cat-tuning`, `cat-voicing`

**Level badge classes:** `level-beginner` (Introductory), `level-foundational`, `level-intermediate`, `level-advanced`, `level-all` (Everyone)

**CSS custom properties** (defined in `:root`): `--bg`, `--card`, `--text`, `--muted`, `--border`, `--accent`, `--accent-light`, `--shadow`

## Key Functions

- `catClass(cat)` — maps a category string to its `cat-*` CSS class
- `classKey(c)` — generates a stable `localStorage` key (`"day|time|title"`) for bookmarking
- `getLevel(c)` / `getDesc(c)` — look up metadata from `classInfo` by title
- `renderAll()` — re-renders all day sections; called on any filter change
- `renderMySchedule()` — renders the "My Schedule" tab
- `toggleSelection(c)` — adds/removes a class from bookmarks; conflict-aware: selecting a class auto-removes any previously bookmarked class in the same time slot

## Persistence

User bookmarks are stored in `localStorage` under the key `ptg2026sel` as a JSON array of `classKey` strings.

## Adding or Updating Classes

1. Add an entry to `schedule[]` with the required fields
2. Optionally add a matching entry to `classInfo{}` for level/description metadata (key must exactly match the `title` field)
3. If introducing a new category, add a `cat-*` CSS rule block in the style section following the existing pattern

## Updating from a New Bubble Schedule PDF

PTG periodically publishes a revised "Bubble Schedule" PDF (linked from a Squarespace URL). The schedule data here is transcribed from it. To sync to a new revision:

1. **Read the PDF, don't WebFetch it.** `WebFetch` cannot parse the PDF (it's image/binary) — but it *does* save the binary to a local path, which it reports in its output. Pass that saved path to the `Read` tool, which renders the pages visually as a grid. (Alternatively, ask the user for the downloaded file path.)
2. **Understand the grid.** Rows are classes grouped by category; columns are day + period with a marker in each scheduled slot. Column order: **Wed P1–5, Thu P1–5, Fri P1–5, Sat P1–4** (Saturday has no P5). Period → `time` mapping:
   | Period | time |
   |---|---|
   | P1 | `8:00–9:30` |
   | P2 | `10:30–12:00` |
   | P3 | `1:30–3:00` |
   | P4 | `3:45–5:15` |
   | P5 | `5:30–7:00` (Wed–Fri only) |
3. **Read the marker shapes.** Filled circle (●) = single-period class. An oval spanning two adjacent columns = 2-period class → `multi: true`, `time` = `"<start> to <next>"` (e.g. `"8:00–9:30 to 10:30–12:00"`). A long oval spanning four columns (P1–P4) = all-day class → `multi: true`, full range. Each row also lists presenter (after `|`) and `Room`.
4. **Diff, don't re-transcribe.** Most classes are unchanged between revisions. Walk the PDF category-by-category against the existing `schedule[]` and only edit what actually differs (time/room/presenter/title, or add/remove). This is far faster and less error-prone than rebuilding the array.
5. **Keep `classInfo{}` in sync with title changes.** If a class is renamed, rename its `classInfo` key to match (descriptions usually still apply). Remove `classInfo` entries for dropped classes. New classes may have no `classInfo` — that's fine (they fall back to level "Everyone", no description).
6. **Update the revision stamp in two places:** the source link in the body (`Schedule source:` line) and the footer link — both the `href` (new Squarespace URL) and the visible `Rev. M/DD/YYYY` label.

Notes: a local copy of the source PDF may live in the repo root for archival, but it is **not** referenced at runtime — the source links point to the remote Squarespace URL. Some entries (e.g. forums with off-grid times like `9:30–10:30am`) may not map cleanly to a period; existing ones use a `"Special"` day with an explanatory `note`.
