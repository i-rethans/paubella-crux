# Custom Crux Puzzle — Design Spec

A personalized NRC-style Crux crossword puzzle, built as a 5-year anniversary gift.

## Context

The Crux is a daily crossword puzzle from the Dutch NRC newspaper. The recipient and creator solve these together daily. This project recreates the digital puzzle experience as a single self-contained HTML file with one hardcoded custom puzzle, personalized theming, and a surprise anniversary message on completion.

## Tech Stack

- **Single HTML file** — vanilla HTML, CSS, and JavaScript. No framework, no build step, no dependencies.
- **Hosting:** GitHub Pages (static file push)
- **Target device:** Desktop/laptop browser with keyboard input
- **Persistence:** Browser localStorage for saving progress between sessions

## Puzzle Format

- Standard crossword grid, approximately 14 columns × 12 rows
- Black cells block off words
- Numbers in the first cell of each word
- Any horizontal or vertical sequence of 2+ letters has an associated clue
- Clues are in Dutch

## Puzzle Data Structure

The puzzle is defined as a hardcoded JavaScript object. A dummy puzzle is used during development; real data is swapped in during Stage 4.

```javascript
const puzzle = {
  rows: 12,
  cols: 14,
  // 2D array: null = black cell, "" = empty white cell
  grid: [
    [null, "", "", "", null, "", "", ...],
    ...
  ],
  // Correct answers for validation (same shape, letters instead of "")
  solution: [
    [null, "W", "O", "R", null, "D", "A", ...],
    ...
  ],
  // Clue number positions and their clues
  clues: {
    horizontal: [
      { number: 1, row: 0, col: 1, length: 3, clue: "Omschrijving" },
      ...
    ],
    vertical: [
      { number: 1, row: 0, col: 1, length: 4, clue: "Omschrijving" },
      ...
    ]
  }
};
```

## Layout

```
┌─────────────────────────────────────────────────────┐
│  [Header / Title area]                              │
├───────────────────┬─────────────────────────────────┤
│                   │  Horizontaal    │  Verticaal     │
│                   │                 │                │
│   Crossword       │  1. clue text   │  1. clue text  │
│   Grid            │  5. clue text   │  2. clue text  │
│                   │  8. clue text   │  3. clue text  │
│                   │  ...            │  ...           │
│                   │                 │                │
│                   │                 │                │
└───────────────────┴─────────────────┴────────────────┘
```

- Grid on the left
- Two clue columns on the right: "Horizontaal" and "Verticaal"

## Interaction Model

### Selection

1. **Click a cell** → selects the word in the *current direction mode* passing through that cell. Cursor jumps to the **first empty cell** in that word. Default direction on page load is horizontal.
2. **Click the same cell again** → toggles the direction mode (horizontal ↔ vertical). Selects the word in the new direction, cursor to first empty cell.
3. **Direction is sticky** — clicking a different cell keeps the current direction mode and selects the word in that direction through the new cell.
4. **Click a clue** in the right panel → selects that word in the grid and sets the direction accordingly (horizontal clue → horizontal mode, vertical clue → vertical mode). Cursor to first empty cell.

### Typing

- Letters fill along the current direction, **skipping cells that already contain a letter**.
- When the end of the word is reached, **wraps around** to the beginning of the word and continues filling empty cells.
- Only uppercase A-Z input is accepted.

### Deletion

- Backspace removes the letter in the current cell (or the previous filled cell in the current direction).
- Deletion follows the selected direction.
- **Wraps around** when the beginning/end of the word is reached.

### Highlighting

- The active word is visually highlighted in the grid (e.g., light blue background).
- The active cell (cursor position) has a distinct highlight (e.g., darker blue).
- The corresponding clue in the right panel is highlighted when a word is selected.

## Completion & Anniversary Surprise

- When all cells are filled, the app checks the entered letters against the solution.
- If all answers are correct, a celebration is triggered:
  - Animation or visual effect (confetti, fade-in, etc.)
  - A personal anniversary message is revealed
  - The specific message content will be provided by the creator
- If answers are incorrect (all cells filled but some wrong), no celebration — the player continues.

## Persistence

- On every letter entry or deletion, the current grid state is saved to `localStorage`.
- On page load, if saved state exists, it is restored.
- Simple JSON serialization of the grid's current letter values.

## Visual Style

- Based on the NRC puzzle layout (clean, structured)
- Personalized with a custom color scheme and/or anniversary theming
- Specific visual customizations to be decided during Stage 6

## Implementation Stages

Each stage is designed to be self-contained, with enough context in this spec to start a fresh session. Each stage gets its own brainstorming session for detailed planning before implementation.

### Stage 1: Grid Rendering & Data Model

**Goal:** Render the crossword grid from hardcoded (dummy) puzzle data.

**Scope:**
- Define the puzzle data structure in JavaScript
- Create a dummy puzzle (~8×8 or smaller) with real Dutch words for development
- Render the grid as CSS grid or HTML table
- Black cells rendered as filled/dark squares
- White cells rendered as input-ready squares
- Clue numbers displayed in the top-left corner of numbered cells
- Basic page structure (HTML skeleton, CSS reset)

**Deliverable:** A page showing a correctly rendered empty crossword grid with numbered cells and black squares. No interaction.

**Context for fresh session:** This spec document. No prior code exists.

### Stage 2: Selection & Navigation Logic

**Goal:** Implement the full keyboard and mouse interaction model.

**Scope:**
- Click-to-select with sticky direction mode
- Direction toggle on re-clicking the same cell
- Cursor placement at first empty cell in the selected word
- Keyboard letter input along the current direction, skipping filled cells, wrapping at word boundaries
- Backspace deletion along the current direction with wrapping
- Visual highlighting of the active word and active cell
- Word boundary detection (finding which word a cell belongs to in a given direction)

**Deliverable:** A fully interactive grid — click cells, type letters, delete, navigate. No clue panel yet.

**Context for fresh session:** This spec document + Stage 1's completed code (single HTML file with grid rendering).

### Stage 3: Clues Panel & Clue-Grid Linking

**Goal:** Add clue columns and bidirectional linking with the grid.

**Scope:**
- Two-column layout beside the grid: "Horizontaal" and "Verticaal"
- Render all clues with their numbers
- Clicking a clue selects the corresponding word in the grid (and sets direction)
- Selecting a word in the grid highlights the corresponding clue
- Scrolling: if clue list is long, the active clue scrolls into view
- CSS layout: grid left, clues right, clean alignment

**Deliverable:** Full puzzle layout with grid and clues working together.

**Context for fresh session:** This spec document + completed code from Stages 1-2.

### Stage 4: Puzzle Data Integration

**Goal:** Replace dummy data with the real custom puzzle.

**Scope:**
- Receive the actual puzzle grid, clues, and solution from the creator
- Integrate into the data structure
- Verify grid renders correctly at actual dimensions (~14×12)
- Verify all clue numbers match, all words have clues, no gaps
- Test edge cases specific to the real grid (short words, adjacent words, etc.)
- Fix any layout issues caused by the larger grid size

**Deliverable:** The real puzzle renders and plays correctly.

**Context for fresh session:** This spec document + completed code from Stages 1-3 + puzzle data provided by the creator.

### Stage 5: Completion Detection & Anniversary Surprise

**Goal:** Detect correct completion and reveal the personal message.

**Scope:**
- On each letter entry, check if all white cells are filled
- When fully filled, compare against the solution
- If correct: trigger celebration (animation, message reveal)
- Design the celebration moment (confetti, overlay, fade-in — specifics decided during this stage's brainstorm)
- Personal message content provided by the creator

**Deliverable:** Solving the puzzle correctly triggers the anniversary surprise.

**Context for fresh session:** This spec document + completed code from Stages 1-4.

### Stage 6: Polish, Persistence & Deployment

**Goal:** Final touches, save/restore, theming, and go live.

**Scope:**
- localStorage save on every change, restore on page load
- Personalized visual theme (colors, fonts, anniversary touches)
- Final cross-browser testing
- GitHub Pages deployment setup
- Final playthrough to verify the full experience end-to-end

**Deliverable:** A live URL ready to share.

**Context for fresh session:** This spec document + completed code from Stages 1-5.

## Tooling Requirements

- **For Claude:** Only a browser to open the HTML file and verify rendering/interaction. No Node.js, no extensions, no build tools needed.
- **For the creator:** A text editor and git. GitHub account for Pages deployment.
