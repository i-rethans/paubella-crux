# Stage 5: Completion Detection, Anniversary Surprise & Arrow Navigation

## Context

Stages 1–4 are complete. The puzzle renders a 12×13 grid with real Dutch clues, supports click-to-select, keyboard letter input/deletion, direction toggling, and bidirectional clue–grid linking. All code lives in a single `index.html` file (vanilla HTML/CSS/JS, no dependencies).

This stage adds three features: detecting when the puzzle is solved correctly, showing a celebration with a personal message, and arrow key navigation to browse between words.

## 1. Completion Detection

### Trigger

After every `placeLetter()` call, invoke `checkCompletion()`.

### Logic

1. **All filled?** — Iterate `state.gridLetters`. For every cell where `puzzle.grid[r][c] !== null`, check that `state.gridLetters[r][c] !== ''`. If any white cell is empty, return early.
2. **All correct?** — Compare every white cell: `state.gridLetters[r][c] === puzzle.solution[r][c]`. If all match, the puzzle is solved.
3. **Wrong answers** — If all cells are filled but some don't match the solution, do nothing. The player continues editing. No error feedback is shown (matches NRC Crux behavior).

### Performance

The grid is 12×13 = 156 cells. A full scan on every keystroke is negligible.

## 2. Celebration: Confetti + Overlay

### Trigger Flow

1. `checkCompletion()` returns true
2. Set `state.completed = true` (disables further input)
3. Wait 300ms (let the player see their final letter land)
4. Launch confetti
5. After 500ms, fade in the overlay message

### Confetti (CSS-based)

- Generate ~60 `<div>` elements inside a fixed-position full-viewport container (`pointer-events: none`)
- Each particle gets randomized properties:
  - Horizontal position: `left` between 0% and 100%
  - Color: picked from a palette of 6 festive colors (gold `#FFD700`, coral `#FF6B6B`, soft pink `#FFB5C2`, sky blue `#87CEEB`, mint `#98D8AA`, lavender `#C4A7E7`)
  - Size: width 8–12px, height 10–16px
  - Animation duration: 2–4s (randomized for natural stagger)
  - Animation delay: 0–800ms (staggered start)
  - Rotation: random initial angle, rotates 2–4 full turns during fall
- CSS `@keyframes confetti-fall`: translate from `translateY(-20px)` to `translateY(110vh)`, with slight horizontal sway via `translateX` oscillation
- Opacity fades to 0 in the last 25% of the animation
- `animation-fill-mode: forwards` so particles stay invisible after falling
- The confetti container is removed from the DOM 5s after creation (after all animations complete)

### Overlay Message

- A fixed-position full-viewport backdrop: `background: rgba(0, 0, 0, 0.4)`
- Centered card (max-width ~480px):
  - White background, rounded corners, padding, subtle box-shadow
  - Header text: `"Gefeliciteerd!"` (hardcoded, can be changed alongside the message before deployment)
  - Personal message paragraph — **placeholder for now: `"[Persoonlijk bericht hier]"`**, to be replaced with the real message before deployment
- Fade-in via CSS transition: opacity 0 → 1 over 600ms
- The overlay appears 500ms after confetti starts (via `setTimeout`)
- No close button — this is the final state of the puzzle

### Color Notes

The confetti palette was chosen to contrast well against the existing `#f5f5f0` page background and `#1a1a1a` grid borders. The overlay backdrop darkens the page enough to make the white message card stand out.

## 3. Arrow Key Word Navigation

### Left/Right — Cycle Through Words in Current Direction

- Get the ordered clue list for `state.direction` (either `puzzle.clues.horizontal` or `puzzle.clues.vertical`)
- Find the index of `state.selectedWord` in that list (match by number + direction)
- **Right arrow**: select the next word in the list, wrapping to the first word after the last
- **Left arrow**: select the previous word, wrapping to the last word after the first
- On selection: cursor moves to the first empty cell in the new word (or the first cell if the word is fully filled)

### Up/Down — Toggle Direction

- Switch `state.direction` between `'horizontal'` and `'vertical'`
- Look for a word in the new direction that passes through the current cursor cell (`state.cursorCell`)
- If a word exists at that cell in the new direction, select it
- If no word exists there, fall back to the first word in the new direction's clue list
- Cursor moves to first empty cell in the selected word

### No Selection State

If no word is currently selected (e.g., fresh page load before any click), any arrow key press selects the first horizontal word (clue #1 horizontal).

### Key Handling

Arrow key handling is added to the existing `keydown` listener, guarded by `state.completed` (ignored after puzzle is solved). `e.preventDefault()` is called for arrow keys to prevent page scrolling.

## 4. Input Blocking After Completion

- New state field: `state.completed = false` (initialized at startup)
- When `checkCompletion()` detects a correct solution, set `state.completed = true`
- The `keydown` handler returns early if `state.completed` is true
- The grid click handler returns early if `state.completed` is true
- The clue click handler returns early if `state.completed` is true

## 5. Code Changes Summary

All changes are within the existing `index.html` file.

### New CSS

- `@keyframes confetti-fall` — particle fall + sway + rotation animation
- `.confetti-container` — fixed full-viewport overlay for particles
- `.confetti-particle` — individual particle styling
- `.celebration-backdrop` — semi-transparent dark overlay
- `.celebration-card` — centered white message card
- `.celebration-header` / `.celebration-message` — text styles within the card
- Fade-in transition class for the overlay

### New JavaScript

- `checkCompletion()` — called at end of `placeLetter()`, checks all cells filled + all correct
- `showCelebration()` — creates confetti particles, schedules overlay reveal, sets up cleanup
- `state.completed` field — boolean, guards all input handlers

### Modified JavaScript

- `placeLetter()` — add `checkCompletion()` call at the end
- `keydown` handler — add `state.completed` guard at top, add arrow key cases (ArrowLeft, ArrowRight, ArrowUp, ArrowDown)
- Grid click handler — add `state.completed` guard
- Clue click handler — add `state.completed` guard

### Unchanged

- Grid rendering, clue rendering, word lookup, selection logic, deletion logic — all untouched.
