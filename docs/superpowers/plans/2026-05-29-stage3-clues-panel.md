# Stage 3: Clues Panel & Clue-Grid Linking — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add two clue columns ("Horizontaal" and "Verticaal") beside the grid with bidirectional linking — clicking a clue selects the word in the grid, selecting a word in the grid highlights the corresponding clue and scrolls it into view.

**Architecture:** All code lives in the single `index.html` file from Stages 1-2. New HTML adds a `#clues-container` div inside the existing `<main>`. New CSS styles the clue panels and active-clue highlighting. New JavaScript renders clues from `puzzle.clues`, builds a lookup map from clue identity to DOM element, extends `updateHighlighting()` to highlight the active clue, and adds click handlers on clue items. No structural changes to existing code — only additions and one extension to `updateHighlighting()`.

**Tech Stack:** Vanilla HTML, CSS, JavaScript. Browser-only. No test framework — verification is manual in the browser.

---

## File Structure

- **Modify:** `index.html` — the only file. All changes are additions within the existing `<style>` and `<script>` blocks, plus one new HTML element in `<main>`.

### Code Organization Within `<script>`

After all tasks, the script block will be organized as follows (existing code marked with ✓, new code marked with ★):

```
✓ puzzle data object
✓ validation assertions
✓ clue direction tagging
✓ state object
✓ wordLookup constant
★ clueElements map (empty, populated during renderClues)
✓ buildNumberMap()
✓ buildWordLookup()
✓ getCell(), getWordCells(), findWordAt(), firstEmptyCell()
✓ renderGrid()
★ renderClues()
✓★ updateHighlighting() (extended with clue highlighting)
✓ selectWord()
★ selectClue()
✓ placeLetter(), advanceCursor()
✓ deleteLetter()
✓ renderGrid(puzzle) call
★ renderClues(puzzle) call
✓ grid click event listener
✓ keydown event listener
```

---

### Task 1: Clue panel HTML structure and CSS layout

**Files:**
- Modify: `index.html` (add HTML in `<body>`, add CSS in `<style>`)

- [ ] **Step 1: Add the clues container HTML**

Add inside `<main>`, immediately after the `<div id="grid-container"></div>` line (line 99 of current file):

```html
    <div id="clues-container">
      <div class="clues-column">
        <h2>Horizontaal</h2>
        <div id="clues-horizontal" class="clues-list"></div>
      </div>
      <div class="clues-column">
        <h2>Verticaal</h2>
        <div id="clues-vertical" class="clues-list"></div>
      </div>
    </div>
```

- [ ] **Step 2: Add CSS for the clue panels**

Add at the end of the `<style>` block, just before the closing `</style>` tag (after the `.cell-active` rule):

```css
    #clues-container {
      display: flex;
      gap: 2rem;
      max-height: 528px;
      overflow: hidden;
    }

    .clues-column {
      min-width: 200px;
      max-width: 260px;
    }

    .clues-column h2 {
      font-size: 1rem;
      font-weight: 700;
      color: #1a1a1a;
      margin-bottom: 0.75rem;
      text-transform: uppercase;
      letter-spacing: 0.03em;
    }

    .clues-list {
      max-height: 496px;
      overflow-y: auto;
      display: flex;
      flex-direction: column;
      gap: 0.25rem;
    }

    .clue-item {
      display: flex;
      gap: 0.5rem;
      padding: 0.35rem 0.5rem;
      border-radius: 3px;
      cursor: pointer;
      font-size: 0.9rem;
      line-height: 1.4;
      color: #333;
    }

    .clue-item:hover {
      background: #eee;
    }

    .clue-number {
      font-weight: 700;
      min-width: 1.5em;
      color: #1a1a1a;
    }

    .clue-active {
      background: #c8e0f4;
    }

    .clue-active:hover {
      background: #c8e0f4;
    }
```

The `max-height: 528px` on `#clues-container` matches the grid height (12 rows × 44px = 528px for the real puzzle). The `overflow-y: auto` on `.clues-list` enables scrolling when clues overflow.

- [ ] **Step 3: Verify in browser**

Run: `open index.html` (or refresh)

Expected:
- The "Horizontaal" and "Verticaal" headings appear to the right of the grid
- The clue lists are empty (not yet rendered by JS)
- The grid still renders and functions correctly (no regression)
- Layout is: grid on the left, two empty clue columns on the right
- No console errors

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add clue panel HTML structure and CSS layout"
```

---

### Task 2: Render clues from puzzle data

**Files:**
- Modify: `index.html` (add JS inside `<script>` block)

- [ ] **Step 1: Add the clueElements lookup map**

Add immediately after the `const wordLookup = buildWordLookup(puzzle);` line (line 171 of current file), before the `function buildWordLookup` definition:

```javascript
    const clueElements = {};
```

This map will be populated during `renderClues()`. Keys are `"horizontal-1"`, `"vertical-2"`, etc. Values are the corresponding DOM elements.

- [ ] **Step 2: Add the renderClues function**

Add after the `renderGrid` function (after its closing `}`), before the `updateHighlighting` function:

```javascript
    function renderClues(puzzle) {
      const hContainer = document.getElementById('clues-horizontal');
      const vContainer = document.getElementById('clues-vertical');

      for (const clue of puzzle.clues.horizontal) {
        const item = document.createElement('div');
        item.classList.add('clue-item');
        item.dataset.direction = 'horizontal';
        item.dataset.number = clue.number;

        const num = document.createElement('span');
        num.classList.add('clue-number');
        num.textContent = clue.number + '.';

        const text = document.createElement('span');
        text.classList.add('clue-text');
        text.textContent = clue.clue;

        item.appendChild(num);
        item.appendChild(text);
        hContainer.appendChild(item);
        clueElements[`horizontal-${clue.number}`] = item;
      }

      for (const clue of puzzle.clues.vertical) {
        const item = document.createElement('div');
        item.classList.add('clue-item');
        item.dataset.direction = 'vertical';
        item.dataset.number = clue.number;

        const num = document.createElement('span');
        num.classList.add('clue-number');
        num.textContent = clue.number + '.';

        const text = document.createElement('span');
        text.classList.add('clue-text');
        text.textContent = clue.clue;

        item.appendChild(num);
        item.appendChild(text);
        vContainer.appendChild(item);
        clueElements[`vertical-${clue.number}`] = item;
      }
    }
```

Each clue item gets stored in `clueElements` for O(1) access when highlighting.

- [ ] **Step 3: Call renderClues on page load**

Add immediately after the `renderGrid(puzzle);` call (line 354 of current file), before the click event listener:

```javascript
    renderClues(puzzle);
```

- [ ] **Step 4: Verify in browser**

Run: `open index.html` (or refresh)

Expected:
- Under "Horizontaal": three clues appear — "1. Huisdier dat spint", "3. Gebied met veel bomen", "4. Zuivelproduct uit Gouda"
- Under "Verticaal": two clues appear — "1. Gebak, vaak met glazuur", "2. Restant na verbranding"
- Clue items show a hover effect (light gray background on mouseover)
- Clicking clues does nothing yet (no click handler added)
- Grid still functions correctly
- No console errors
- Verify in console: `clueElements["horizontal-1"]` returns a DOM element, `clueElements["vertical-2"]` returns a DOM element

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: render clues from puzzle data with lookup map"
```

---

### Task 3: Bidirectional linking — clue click selects grid word

**Files:**
- Modify: `index.html` (add JS in `<script>` block)

- [ ] **Step 1: Add the selectClue function**

Add after the `selectWord` function, before the `placeLetter` function:

```javascript
    function selectClue(clue) {
      state.direction = clue.direction;
      state.selectedWord = clue;
      state.cursorCell = firstEmptyCell(clue, { row: clue.row, col: clue.col });
      state.lastClickedCell = null;
      updateHighlighting();
    }
```

This is simpler than `selectWord` — no direction toggle or fallback logic needed. We know exactly which clue was clicked. Setting `lastClickedCell` to `null` ensures the next grid click won't trigger a direction toggle.

- [ ] **Step 2: Add click handlers to clue items**

Add immediately after the `renderClues(puzzle);` call, before the grid click event listener:

```javascript
    document.getElementById('clues-container').addEventListener('click', (e) => {
      const item = e.target.closest('.clue-item');
      if (!item) return;
      const direction = item.dataset.direction;
      const number = parseInt(item.dataset.number);
      const clue = puzzle.clues[direction].find(c => c.number === number);
      if (clue) selectClue(clue);
    });
```

Uses event delegation on the container, same pattern as the grid click handler. Finds the clue object by matching direction and number from the DOM element's data attributes.

- [ ] **Step 3: Verify in browser**

Run: `open index.html` (or refresh)

Test these scenarios:

1. **Click clue "1. Huisdier dat spint"** (horizontal) — grid highlights cells (0,1), (0,2), (0,3) in light blue. Cursor (darker blue) at (0,1). Direction is now horizontal.

2. **Click clue "1. Gebak, vaak met glazuur"** (vertical) — grid highlights cells (0,1), (1,1), (2,1), (3,1). Cursor at (0,1). Direction is now vertical.

3. **Click clue "4. Zuivelproduct uit Gouda"** (horizontal) — grid highlights cells (3,1), (3,2), (3,3), (3,4). Cursor at (3,1). Direction is now horizontal.

4. **Type "KAT" after clicking horizontal clue 1** — letters fill the grid. Click vertical clue 1 — cursor goes to (2,1) (skips filled cells at (0,1) and (1,1) if they have letters — but (1,1) is empty so cursor goes to (0,1)).

5. **Click a grid cell after clicking a clue** — normal grid selection behavior resumes. No direction toggle on first click (since `lastClickedCell` was cleared).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add clue click handlers to select words in grid"
```

---

### Task 4: Bidirectional linking — grid selection highlights active clue

**Files:**
- Modify: `index.html` (modify `updateHighlighting` function in `<script>` block)

- [ ] **Step 1: Extend updateHighlighting to highlight the active clue**

Find the existing `updateHighlighting` function. Replace the entire function with:

```javascript
    function updateHighlighting() {
      document.querySelectorAll('.cell-highlight, .cell-active').forEach(el => {
        el.classList.remove('cell-highlight', 'cell-active');
      });

      const prevActive = document.querySelector('.clue-active');
      if (prevActive) prevActive.classList.remove('clue-active');

      if (!state.selectedWord) return;

      const cells = getWordCells(state.selectedWord);
      for (const { row, col } of cells) {
        getCell(row, col).classList.add('cell-highlight');
      }

      if (state.cursorCell) {
        const active = getCell(state.cursorCell.row, state.cursorCell.col);
        active.classList.remove('cell-highlight');
        active.classList.add('cell-active');
      }

      const key = `${state.selectedWord.direction}-${state.selectedWord.number}`;
      const clueEl = clueElements[key];
      if (clueEl) {
        clueEl.classList.add('clue-active');
        clueEl.scrollIntoView({ block: 'nearest', behavior: 'smooth' });
      }
    }
```

Changes from the original:
1. Clears the previous `.clue-active` class before applying highlights.
2. After grid highlighting, looks up the active clue's DOM element via `clueElements` and adds `.clue-active`.
3. Calls `scrollIntoView({ block: 'nearest', behavior: 'smooth' })` to ensure the active clue is visible when the list is scrollable.

- [ ] **Step 2: Verify in browser**

Run: `open index.html` (or refresh)

Test these scenarios:

1. **Click cell (0,1)** — horizontal word "KAT" highlights in grid. In the "Horizontaal" column, clue "1. Huisdier dat spint" gets a light blue background.

2. **Click cell (0,1) again** — toggles to vertical. Word "KOEK" highlights in grid. In the "Verticaal" column, clue "1. Gebak, vaak met glazuur" gets a light blue background. The previous horizontal clue loses its highlight.

3. **Click cell (1,0)** — horizontal "BOS" highlights in grid. Clue "3. Gebied met veel bomen" highlights. Previous clue deactivated.

4. **Click clue "4. Zuivelproduct uit Gouda"** — word highlights in grid AND the clue itself is highlighted (blue background). Direction is horizontal.

5. **Click a different grid cell** — clue highlight moves to match the newly selected word.

6. **Deselection check:** At no point should two clues be highlighted simultaneously.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: highlight active clue in panel when grid word is selected"
```

---

## Verification Checklist

After all tasks are complete, perform a full end-to-end test:

- [ ] Page loads without console errors
- [ ] Console prints "Puzzle data validated OK"
- [ ] Grid renders correctly (same as Stage 2)
- [ ] **Clue rendering:** "Horizontaal" column shows 3 clues (1, 3, 4) with correct Dutch text
- [ ] **Clue rendering:** "Verticaal" column shows 2 clues (1, 2) with correct Dutch text
- [ ] **Layout:** Grid on the left, clue columns on the right, clean alignment
- [ ] **Clue → Grid (click clue):** Clicking a horizontal clue selects the corresponding word in the grid, sets direction to horizontal, cursor at first empty cell
- [ ] **Clue → Grid (click clue):** Clicking a vertical clue selects the corresponding word in the grid, sets direction to vertical, cursor at first empty cell
- [ ] **Grid → Clue (click grid):** Clicking a grid cell highlights the corresponding clue in the right panel
- [ ] **Grid → Clue (toggle):** Toggling direction on a grid cell switches the highlighted clue
- [ ] **Active clue styling:** Exactly one clue is highlighted at any time (light blue background matching grid highlight)
- [ ] **Scroll into view:** Active clue scrolls into view (testable with real puzzle in Stage 4 — verify no errors for now)
- [ ] **Hover effect:** Clue items show a subtle hover effect
- [ ] **No regression — typing:** Letter input still works correctly with word wrapping and skip-filled behavior
- [ ] **No regression — deletion:** Backspace still works with backward search and wrap
- [ ] **No regression — direction toggle:** Re-clicking the same grid cell still toggles direction
- [ ] **No regression — black cells:** Clicking black cells still does nothing
