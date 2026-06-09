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
