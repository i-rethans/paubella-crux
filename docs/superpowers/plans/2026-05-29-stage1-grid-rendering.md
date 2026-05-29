# Stage 1: Grid Rendering & Data Model — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Render a crossword grid from hardcoded dummy puzzle data as a single self-contained HTML file — black cells, white cells, and clue numbers all correctly placed.

**Architecture:** Single `index.html` file containing all HTML, CSS, and JavaScript inline. No framework, no build step, no dependencies. The puzzle is defined as a JavaScript object; a rendering function reads it and builds the DOM. CSS Grid handles the cell layout.

**Tech Stack:** Vanilla HTML, CSS, JavaScript. Browser-only — no Node.js, no test framework. Verification is visual (open in browser).

---

## File Structure

- **Create:** `index.html` — the entire application. Contains:
  - HTML skeleton with header and grid container
  - `<style>` block with CSS reset, layout, and grid cell styles
  - `<script>` block with puzzle data object and rendering function

---

## Dummy Puzzle Design

A 5×5 grid with 5 Dutch words for development. This grid exercises all rendering features: black cells, white cells, clue numbers, and cells where horizontal and vertical words cross.

```
     0    1    2    3    4
  ┌────┬────┬────┬────┬────┐
0 │ ██ │ K¹ │ A² │ T  │ ██ │
  ├────┼────┼────┼────┼────┤
1 │ B³ │ O  │ S  │ ██ │ ██ │
  ├────┼────┼────┼────┼────┤
2 │ ██ │ E  │ ██ │ ██ │ ██ │
  ├────┼────┼────┼────┼────┤
3 │ ██ │ K⁴ │ A  │ A  │ S  │
  ├────┼────┼────┼────┼────┤
4 │ ██ │ ██ │ ██ │ ██ │ ██ │
  └────┴────┴────┴────┴────┘
```

**Words:**

| #  | Direction   | Word | Clue (Dutch)                 |
|----|-------------|------|------------------------------|
| 1  | Horizontaal | KAT  | Huisdier dat spint           |
| 3  | Horizontaal | BOS  | Gebied met veel bomen        |
| 4  | Horizontaal | KAAS | Zuivelproduct uit Gouda      |
| 1  | Verticaal   | KOEK | Gebak, vaak met glazuur      |
| 2  | Verticaal   | AS   | Restant na verbranding       |

**Numbering logic** (standard crossword rules — scan left→right, top→bottom):
- Cell (0,1): left is black → H start. Top is edge → V start. **Number 1.**
- Cell (0,2): left is white → no H. Top is edge, below is white → V start. **Number 2.**
- Cell (1,0): left is edge → H start. Above is black, below is black → no V. **Number 3.**
- Cell (3,1): left is black → H start. Above is white → no V. **Number 4.**

---

### Task 1: HTML skeleton and CSS foundation

**Files:**
- Create: `index.html`

- [ ] **Step 1: Create index.html with doctype, head, and empty body**

```html
<!DOCTYPE html>
<html lang="nl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Crux</title>
  <style>
    *, *::before, *::after {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      font-family: system-ui, -apple-system, sans-serif;
      background: #f5f5f0;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 2rem;
    }
  </style>
</head>
<body>
</body>
</html>
```

- [ ] **Step 2: Add header and grid container to body**

Add inside `<body>`:

```html
<header>
  <h1>Crux</h1>
</header>
<main>
  <div id="grid-container"></div>
</main>
```

Add to the `<style>` block:

```css
header {
  margin-bottom: 1.5rem;
}

h1 {
  font-size: 2rem;
  font-weight: 700;
  letter-spacing: 0.05em;
  color: #1a1a1a;
}

main {
  display: flex;
  gap: 2rem;
}
```

- [ ] **Step 3: Open in browser and verify**

Run: `open index.html` (or refresh if already open)

Expected: A page with off-white background, "Crux" heading centered at the top, nothing else visible. No console errors.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add HTML skeleton with CSS reset and page layout"
```

---

### Task 2: Define puzzle data structure with dummy data

**Files:**
- Modify: `index.html` (add `<script>` block)

- [ ] **Step 1: Add the puzzle data object**

Add a `<script>` block before `</body>` in `index.html`:

```html
<script>
const puzzle = {
  rows: 5,
  cols: 5,
  grid: [
    [null, "",   "",   "",   null],
    ["",   "",   "",   null, null],
    [null, "",   null, null, null],
    [null, "",   "",   "",   ""  ],
    [null, null, null, null, null],
  ],
  solution: [
    [null, "K", "A", "T", null],
    ["B",  "O", "S", null, null],
    [null, "E", null, null, null],
    [null, "K", "A", "A", "S"],
    [null, null, null, null, null],
  ],
  clues: {
    horizontal: [
      { number: 1, row: 0, col: 1, length: 3, clue: "Huisdier dat spint" },
      { number: 3, row: 1, col: 0, length: 3, clue: "Gebied met veel bomen" },
      { number: 4, row: 3, col: 1, length: 4, clue: "Zuivelproduct uit Gouda" },
    ],
    vertical: [
      { number: 1, row: 0, col: 1, length: 4, clue: "Gebak, vaak met glazuur" },
      { number: 2, row: 0, col: 2, length: 2, clue: "Restant na verbranding" },
    ],
  },
};
</script>
```

- [ ] **Step 2: Add validation assertions**

Add immediately after the `puzzle` object, inside the same `<script>` block:

```javascript
console.assert(puzzle.grid.length === puzzle.rows,
  `Grid has ${puzzle.grid.length} rows, expected ${puzzle.rows}`);
console.assert(puzzle.grid.every(row => row.length === puzzle.cols),
  `Not all grid rows have ${puzzle.cols} columns`);
console.assert(puzzle.solution.length === puzzle.rows,
  `Solution has ${puzzle.solution.length} rows, expected ${puzzle.rows}`);
console.assert(puzzle.solution.every(row => row.length === puzzle.cols),
  `Not all solution rows have ${puzzle.cols} columns`);
for (let r = 0; r < puzzle.rows; r++) {
  for (let c = 0; c < puzzle.cols; c++) {
    const isBlackGrid = puzzle.grid[r][c] === null;
    const isBlackSolution = puzzle.solution[r][c] === null;
    console.assert(isBlackGrid === isBlackSolution,
      `Black cell mismatch at (${r},${c})`);
  }
}
console.log("Puzzle data validated OK");
```

- [ ] **Step 3: Open in browser and verify**

Run: `open index.html` (or refresh)

Expected: Browser console shows `"Puzzle data validated OK"`. No assertion failures.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add dummy puzzle data structure with validation"
```

---

### Task 3: Render the grid with black cells, white cells, and clue numbers

**Files:**
- Modify: `index.html` (add CSS for grid cells, add rendering JS)

- [ ] **Step 1: Add CSS for the grid and cells**

Add to the `<style>` block:

```css
#grid-container {
  display: grid;
  gap: 0;
  border: 2px solid #1a1a1a;
  background: #1a1a1a;
  width: fit-content;
}

.cell {
  width: 44px;
  height: 44px;
  position: relative;
  border: 1px solid #1a1a1a;
}

.cell-black {
  background: #1a1a1a;
}

.cell-white {
  background: #ffffff;
}

.cell-number {
  position: absolute;
  top: 1px;
  left: 2px;
  font-size: 10px;
  line-height: 1;
  color: #1a1a1a;
  pointer-events: none;
}

.cell-letter {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  font-size: 20px;
  font-weight: 500;
  color: #1a1a1a;
  text-transform: uppercase;
}
```

- [ ] **Step 2: Build the number map from clues data**

Add to the `<script>` block, after the validation code:

```javascript
function buildNumberMap(clues) {
  const map = {};
  for (const clue of [...clues.horizontal, ...clues.vertical]) {
    map[`${clue.row},${clue.col}`] = clue.number;
  }
  return map;
}
```

- [ ] **Step 3: Write the grid rendering function**

Add to the `<script>` block:

```javascript
function renderGrid(puzzle) {
  const container = document.getElementById("grid-container");
  container.style.gridTemplateColumns = `repeat(${puzzle.cols}, 44px)`;
  container.style.gridTemplateRows = `repeat(${puzzle.rows}, 44px)`;

  const numberMap = buildNumberMap(puzzle.clues);

  for (let r = 0; r < puzzle.rows; r++) {
    for (let c = 0; c < puzzle.cols; c++) {
      const cell = document.createElement("div");
      cell.classList.add("cell");
      cell.dataset.row = r;
      cell.dataset.col = c;

      if (puzzle.grid[r][c] === null) {
        cell.classList.add("cell-black");
      } else {
        cell.classList.add("cell-white");

        const number = numberMap[`${r},${c}`];
        if (number !== undefined) {
          const numSpan = document.createElement("span");
          numSpan.classList.add("cell-number");
          numSpan.textContent = number;
          cell.appendChild(numSpan);
        }

        const letterSpan = document.createElement("span");
        letterSpan.classList.add("cell-letter");
        cell.appendChild(letterSpan);
      }

      container.appendChild(cell);
    }
  }
}
```

- [ ] **Step 4: Call the render function on page load**

Add at the end of the `<script>` block:

```javascript
renderGrid(puzzle);
```

- [ ] **Step 5: Open in browser and verify**

Run: `open index.html` (or refresh)

Expected:
- A 5×5 grid is visible below the "Crux" heading
- Black cells (8 of them) are dark/filled — verify positions: (0,0), (0,4), (1,3), (1,4), (2,0), (2,2), (2,3), (2,4) — and the entire bottom row
- White cells (11 of them) are white with borders
- Number "1" appears top-left in cell (0,1)
- Number "2" appears top-left in cell (0,2)
- Number "3" appears top-left in cell (1,0)
- Number "4" appears top-left in cell (3,1)
- No other cells have numbers
- All white cells are empty (no letters)
- Console shows "Puzzle data validated OK" and no errors

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: render crossword grid with black cells, white cells, and clue numbers"
```

---

## Verification Checklist

After all tasks are complete, verify the full Stage 1 deliverable:

- [ ] Page loads without console errors
- [ ] Console prints "Puzzle data validated OK"
- [ ] Grid renders at correct dimensions (5 columns × 5 rows)
- [ ] Black cells are visually distinct (dark fill)
- [ ] White cells are empty and have clear borders
- [ ] Exactly 4 cells show clue numbers (1, 2, 3, 4) in the correct positions
- [ ] Numbers appear small in the top-left corner of their cells
- [ ] Grid is centered on the page below the "Crux" heading
- [ ] No interactivity — clicking does nothing (that's Stage 2)
