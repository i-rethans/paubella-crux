# Stage 6: Polish, Persistence & Deployment

## Context

Stages 1–5 are complete. The puzzle renders a 12×13 grid with real Dutch clues, supports click-to-select, keyboard letter input/deletion, direction toggling, bidirectional clue–grid linking, arrow key word navigation, completion detection, and a confetti + overlay celebration. All code lives in a single `index.html` file (vanilla HTML/CSS/JS, no dependencies).

This stage adds three things: localStorage persistence, a warm anniversary visual theme, and GitHub Pages deployment.

## 1. localStorage Persistence

### Save

After every `placeLetter()` and `deleteLetter()` call, serialize `state.gridLetters` to localStorage.

```javascript
localStorage.setItem('crux-progress', JSON.stringify(state.gridLetters));
```

Key name: `crux-progress`. The value is a JSON 2D array matching the grid dimensions. `null` entries (black cells) are preserved as-is.

### Restore

On page load, after the grid is rendered, check for saved state:

1. Read `localStorage.getItem('crux-progress')`
2. If it exists, parse the JSON and validate: must be an array with `puzzle.rows` rows, each with `puzzle.cols` columns
3. For each cell where `puzzle.grid[r][c] !== null` and the saved value is a single uppercase letter A–Z, restore it:
   - Set `state.gridLetters[r][c]` to the saved letter
   - Set the corresponding `.cell-letter` element's `textContent`
4. After restoring, run `checkCompletion()` in case the puzzle was already solved — this handles the edge case where the user solved it, closed the tab before the celebration rendered, and reopened

### No Reset Button

There is no UI to clear progress. The user can clear localStorage manually via DevTools if needed. This is a one-time gift puzzle, not a replayable game.

## 2. Visual Theme: Warm & Personal

Shift the entire color palette from cool/neutral to warm tones. The goal: feel like a handcrafted gift, not a newspaper app. All changes are CSS-only — no structural HTML changes.

### Color Palette

| Element | Current | New |
|---|---|---|
| Page background | `#f5f5f0` | `#faf7f2` (warm cream) |
| Grid borders & black cells | `#1a1a1a` | `#2c2420` (warm dark brown) |
| White cells | `#ffffff` | `#fffdf8` (warm white) |
| Active word highlight | `#c8e0f4` (light blue) | `#f0dfc8` (light tan) |
| Active cell (cursor) | `#7fb8e0` (medium blue) | `#d4b896` (warm gold) |
| Clue active highlight | `#c8e0f4` | `#f0dfc8` |
| Clue hover | `#eee` | `#f0ebe0` |
| Header text | `#1a1a1a` | `#2c2420` |
| Body text / clue text | `#333` | `#4a3f35` |
| Cell numbers | `#1a1a1a` | `#2c2420` |
| Cell letters | `#1a1a1a` | `#2c2420` |

### Header Changes

- Font family: add `Georgia, serif` to the h1
- Keep "Crux" as the title
- Add a subtitle element below h1: `⋆ 5 Jaar ⋆`
  - Styled: smaller font, color `#a08060` (muted gold), centered

### Subtitle HTML

Add inside the existing `<header>` element, after the h1:

```html
<p class="subtitle">⋆ 5 Jaar ⋆</p>
```

### Subtitle CSS

```css
.subtitle {
  font-size: 0.85rem;
  color: #a08060;
  text-align: center;
  margin-top: 0.25rem;
}
```

### Celebration Overlay Update

- Change the overlay header text from `'Gefeliciteerd!'` to `'Puzzel opgelost!'`
- The personal message stays as `'[Persoonlijk bericht hier]'` — to be replaced before deployment
- Overlay card styling updates to match warm palette:
  - Card background stays `#ffffff` (white card on darkened backdrop is fine)
  - Header color: `#2c2420`
  - Message color: `#4a3f35`

### Confetti Palette

Keep the existing confetti colors — they're festive and contrast well against the warm cream background. No change needed.

### What Does NOT Change

- Grid dimensions, cell sizes (44px)
- Layout structure (grid left, clues right)
- Font sizes for cells (20px), clue numbers, clue text
- All interaction logic
- Confetti animation

## 3. Deployment

### GitHub Pages Setup

1. Create a GitHub repository (e.g., `crux-puzzle` or the user's preferred name)
2. Add `index.html` to the repo root
3. Enable GitHub Pages from the repository settings (deploy from main branch, root directory)
4. The live URL will be `https://<username>.github.io/<repo-name>/`

### Pre-deployment Checklist

- Replace `'[Persoonlijk bericht hier]'` with the real personal message
- Final playthrough: solve the puzzle end-to-end, verify celebration triggers
- Verify localStorage save/restore works (partially fill, refresh, check state persists)
- Check in at least Chrome and Safari

### .gitignore

Add `.superpowers/` to `.gitignore` to keep brainstorming artifacts out of the deployed repo.

## 4. Code Changes Summary

All changes are within the existing `index.html` file, plus a new `.gitignore`.

### New File

- `.gitignore` — excludes `.superpowers/`

### Modified CSS

- Update all color values per the palette table above
- Add `Georgia, serif` to h1 font-family
- Add `.subtitle` class for the anniversary badge
- Update `.celebration-header` and `.celebration-message` colors

### New HTML

- Add `<p class="subtitle">⋆ 5 Jaar ⋆</p>` inside `<header>`

### New JavaScript

- `saveProgress()` — serializes `state.gridLetters` to localStorage
- `restoreProgress()` — loads and validates saved state, populates grid, runs `checkCompletion()`

### Modified JavaScript

- `placeLetter()` — call `saveProgress()` after `checkCompletion()`
- `deleteLetter()` — call `saveProgress()` at the end
- `showCelebration()` — change header text to `'Puzzel opgelost!'`
- Initialization: call `restoreProgress()` after `renderGrid()` and `renderClues()`

### Unchanged

- Grid rendering, clue rendering, word lookup, selection logic, deletion logic, completion detection, confetti animation, arrow key navigation — all untouched.
