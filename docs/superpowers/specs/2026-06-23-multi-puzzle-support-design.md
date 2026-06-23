# Multi-Puzzle Support Design

Turn Paubella Crux from a single hardcoded puzzle into a small puzzle collection with a selection page, external puzzle data files, and per-puzzle progress tracking. Still a single `index.html` deployed on GitHub Pages — no framework, no build step.

## Puzzle Data Format

Each puzzle lives in its own JSON file under `puzzles/`.

**File:** `puzzles/{id}.json`

```json
{
  "id": "puzzle-1",
  "title": "5 Jaar",
  "subtitle": "30 mei 2026",
  "description": "Onze eerste kruiswoordpuzzel",
  "grid": ["SLOT##KIND#VI", "CAFE#KAS##DAN", "..."],
  "clues": {
    "horizontal": {
      "SLOT": "Vraag ik altijd over voor we gaan slapen",
      "KIND": "Wil ik heeeel graag"
    },
    "vertical": {
      "SLOT": "Gewenste eigenschap van 20 horizontaal"
    }
  },
  "celebration": {
    "title": "Gefeliciteerd!",
    "message": "5 jaar samen <3"
  }
}
```

**Fields:**

| Field              | Required | Description                                                                                                                   |
| ------------------ | -------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `id`               | yes      | Unique slug, used in URL hash and localStorage keys                                                                           |
| `title`            | yes      | Display title on selection page and puzzle header                                                                             |
| `subtitle`         | yes      | Secondary line (date, occasion, etc.)                                                                                         |
| `description`      | yes      | Short description for the selection page card                                                                                 |
| `grid`             | yes      | Array of strings — letters for solution cells, `#` for black cells. Each string is one row. All rows must be the same length. |
| `clues.horizontal` | yes      | Object mapping answer word (uppercase) to clue text. Numbers are assigned automatically by grid position.                      |
| `clues.vertical`   | yes      | Object mapping answer word (uppercase) to clue text. Numbers are assigned automatically by grid position.                      |
| `celebration`      | no       | Optional object with `title` and `message` for the completion overlay. Falls back to generic "Puzzel opgelost!" if omitted.   |

The grid rows are processed by the existing `parseGrid()` function. Clue numbers and word positions are derived by `detectWords()` and `assignNumbers()`. Clues are keyed by answer word (e.g. `"SLOT"`) rather than by number — the code reads each detected word's letters from the solution grid and looks up the matching clue text. This means you never need to manually figure out clue numbering; just key each clue by the word it describes. The only constraint is that the same word cannot appear twice in the same direction within one puzzle (unlikely for personal puzzles).

## Puzzle Manifest

**File:** `puzzles/index.json`

```json
[
  {
    "id": "puzzle-1",
    "title": "5 Jaar",
    "subtitle": "30 mei 2026",
    "description": "Onze eerste kruiswoordpuzzel",
    "file": "puzzle-1.json"
  },
  {
    "id": "puzzle-2",
    "title": "Veluwe",
    "subtitle": "Fietsen & kamperen",
    "description": "Naar de veluwe!",
    "file": "puzzle-2.json"
  }
]
```

The selection page fetches this single file on load to render puzzle cards — it does not need to load every puzzle JSON just to show the list. The `file` field is relative to `puzzles/`. Title/subtitle/description are duplicated here from the puzzle file on purpose (avoids fetching all puzzles on the selection page). To add a new puzzle: create the JSON file in `puzzles/` and add an entry here.

## Selection Page

Displayed when the URL has no hash (or empty hash). Shows:

- **Header:** "Paubella Crux" (same as current)
- **Puzzle cards:** One per entry in `index.json`, each showing:
  - Title and subtitle
  - Description
  - Progress indicator (e.g. "12/45" or a small progress bar), derived from that puzzle's localStorage data
  - Completed checkmark if the puzzle has been solved
- **Styling:** Same warm anniversary theme (cream background, warm brown text, gold accents)

Clicking a card navigates to `#{puzzle-id}`, which triggers the puzzle view.

## Hash Routing

Single `index.html` with two views in the DOM — selection and puzzle. Only one is visible at a time.

**Routes:**

- No hash / empty hash → selection view
- `#{puzzle-id}` (e.g. `#puzzle-1`) → puzzle view for that puzzle

**Implementation:**

- Listen to `hashchange` and run routing on page load
- Read `location.hash`, strip the `#`
- If empty → show selection, hide puzzle
- If a puzzle ID → hide selection, load and show that puzzle

**Back navigation:**

- Puzzle view header includes a "back to puzzles" link that sets `location.hash = ''`
- Browser back button works automatically (each hash change is a history entry)

## Puzzle Loading

When navigating to `#{id}`:

1. Fetch `puzzles/{id}.json`
2. Pass `grid` rows through `parseGrid()` to get the grid and solution arrays
3. Run `detectWords()` and `assignNumbers()` to find word positions and numbers
4. Match each detected word to its clue by reading the word's letters from the solution grid and looking up the answer in the clue map
5. Assemble the puzzle object (same shape as the current hardcoded one)
6. Call `renderGrid()` and `renderClues()`
7. Restore progress from `localStorage` key `crux-progress-{id}`
8. Initialize state and attach event handlers

The existing parse/render/interaction code stays intact — it just gets called with data from a fetched JSON file instead of inline data.

## localStorage

Per-puzzle keys replace the current single `crux-progress` key:

| Key                   | Value                    | Purpose                                                       |
| --------------------- | ------------------------ | ------------------------------------------------------------- |
| `crux-progress-{id}`  | JSON 2D array of letters | Save/restore grid input                                       |
| `crux-completed-{id}` | `"true"`                 | Selection page shows completed state without parsing the grid |

`saveProgress()` and `restoreProgress()` receive the puzzle ID as a parameter. Same serialization logic, just scoped.

No backwards-compatibility migration for the old `crux-progress` key.

## Celebration

Same confetti animation for all puzzles. The overlay text comes from the puzzle's `celebration` field:

- If `celebration` exists → use `celebration.title` and `celebration.message`
- If omitted → show "Puzzel opgelost!" as title, no message

On completion, also write `crux-completed-{id}` to localStorage so the selection page can show the checkmark without re-checking the full grid.

## Refactoring Scope

The main structural change is extracting the current init flow into reusable functions:

- **Current:** Page load → hardcoded puzzle data → parse → render → attach handlers
- **New:** Page load → route → either show selection (fetch manifest) or load puzzle (fetch JSON → parse → render → attach handlers)

The puzzle engine (grid rendering, selection, navigation, input handling, highlighting, completion checking) stays unchanged. It just gets wrapped in a `loadPuzzle(id)` entry point.

## File Structure

```
crux/
  index.html              # App shell, selection page, puzzle engine, routing
  puzzles/
    index.json            # Manifest of available puzzles
    puzzle-1.json         # First puzzle (extracted from current index.html)
    puzzle-2.json         # Second puzzle (Veluwe trip)
  docs/                   # Specs and plans (unchanged)
  README.md
```
