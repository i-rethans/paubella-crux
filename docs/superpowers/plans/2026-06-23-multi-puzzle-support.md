# Multi-Puzzle Support Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Turn the single-puzzle Paubella Crux app into a multi-puzzle collection with a selection page, external JSON puzzle files, and hash-based routing — all within one `index.html`.

**Architecture:** The existing puzzle engine (parse, render, interact) stays intact. We wrap it in a `loadPuzzle(id)` function that fetches puzzle data from `puzzles/{id}.json`. A hash router (`#puzzle-1`) switches between a selection view and the puzzle view. A manifest file (`puzzles/index.json`) drives the selection page.

**Tech Stack:** Vanilla HTML/CSS/JS, no dependencies, no build step, GitHub Pages deployment.

## Global Constraints

- Single `index.html` file — no framework, no build tools
- All puzzle data in `puzzles/` directory as JSON files
- Must work when served from GitHub Pages (static files only)
- Existing warm anniversary theme (cream `#faf7f2`, brown `#2c2420`, gold `#d4b896`)
- Clues keyed by answer word (uppercase), not by number

---

### Task 1: Extract puzzle-1 data into JSON and create manifest

Extract the hardcoded puzzle data from `index.html` into `puzzles/puzzle-1.json` and create the `puzzles/index.json` manifest. After this task, the files exist but nothing reads them yet.

**Files:**
- Create: `puzzles/puzzle-1.json`
- Create: `puzzles/index.json`

**Produces:**
- `puzzles/puzzle-1.json` — puzzle data with word-keyed clues
- `puzzles/index.json` — manifest array with one entry

- [ ] **Step 1: Create the puzzles directory**

Run: `mkdir -p puzzles`

- [ ] **Step 2: Create puzzle-1.json**

Extract the grid rows and clue texts from `index.html` (lines 478–557). Convert clue keys from numbers to answer words (read the word's letters from the grid rows). The grid rows in `index.html` are:

```
SLOT##KIND#VI
CAFE#KAS##DAN
H###KAAS#MELK
EN#KNAP#ADS#T
RUMOURS##MES#
P#AMSTERDAM##
###B###OE###L
R#UUT##FLINKE
IN#CUE#FLOOR#
O##H###AA#MAM
JAPANS###LAS#
ARA#SIERAAD##
```

To build the word-keyed clues: for each clue in the current code, read the comment next to it (e.g. `// SLOT`) — that's the answer word key. Create `puzzles/puzzle-1.json`:

```json
{
  "id": "puzzle-1",
  "title": "5 Jaar",
  "subtitle": "30 mei 2026",
  "description": "Onze eerste kruiswoordpuzzel",
  "grid": [
    "SLOT##KIND#VI",
    "CAFE#KAS##DAN",
    "H###KAAS#MELK",
    "EN#KNAP#ADS#T",
    "RUMOURS##MES#",
    "P#AMSTERDAM##",
    "###B###OE###L",
    "R#UUT##FLINKE",
    "IN#CUE#FLOOR#",
    "O##H###AA#MAM",
    "JAPANS###LAS#",
    "ARA#SIERAAD##"
  ],
  "clues": {
    "horizontal": {
      "SLOT": "Vraag ik altijd over voor we gaan slapen",
      "KIND": "Wil ik heeeel graag",
      "VI": "Samen met 25 verticaal 👶",
      "CAFE": "Tent met taps",
      "KAS": "Restaurant voor mijn 25ste",
      "DAN": "Past voor ken en sen",
      "KAAS": "Favoriet op de borrelplank",
      "MELK": "Ga ik van scheten",
      "EN": "Jij _ ik",
      "KNAP": "Dat ben jij!",
      "ADS": "Computer Science vak",
      "RUMOURS": "Album covers",
      "MES": "Hebben we elkaar allebei een keer kado gedaan",
      "AMSTERDAM": "Waar ons huis woont",
      "OE": "Jouw favoriete stopwoordje van mij",
      "UUT": "Halve marathon en mijn kantoor",
      "FLINKE": "Goede grootte",
      "IN": "Verbindt vriend en tiem",
      "CUE": "Annivesary resto",
      "FLOOR": "Wij zitten op de vierde",
      "AA": "Gaan we zondag nodig hebben",
      "MAM": "__ Bellen",
      "JAPANS": "Land van oorsprong van jouw 20 horizontaal",
      "LAS": "Joe speedboot activiteit in het verleden",
      "ARA": "Standaard gevraagd in een zweedse puzzel",
      "SIERAAD": "Een van mijn eerste kados van Pig en Hen"
    },
    "vertical": {
      "SCHERP": "Gewenste eigenschap van 20 horizontaal",
      "LA": "Enige over van het jaren 20 tafeltje",
      "OF": "Een keuze",
      "TE": "Nooit goed",
      "KAAPSE": "Karel die een lekker biertje is",
      "ISS": "Ga je heen als je austronouten droom uitkomt",
      "VAL": "Jij raakte je horloge kwijt en ik een stukje van mn kin tijdens een gezamelijke __",
      "INKT": "Past voor vis en patroon",
      "KAART": "Ons huis ligt west op de __ van amsterdam",
      "DESEM": "Fave brood, isa proof en homemade",
      "KNUS": "Ons huisje is _ en gezellig",
      "MDMA": "Maakt ons happy op feestjes",
      "NU": "Wanneer zijn we 5 jaar samen?",
      "KOMBUCHA": "Vind ik stinken maar wel lekker drinken",
      "MA": "Samen met 40 verticaal je ouders",
      "ROFFA": "Van __ en 22 verticaal kwamen we",
      "DELLA": "Van __ en 22 verticaal kwamen we",
      "LE": "Samen met 7 horizontaal 👶",
      "RIOJA": "Fave rode wijn",
      "TU": "Universiteit waar we inginieur zijn geworden",
      "IO": "\"Makkelijke studie\"",
      "NOMAD": "Wij in Parijs",
      "KRAS": "Zat al snel op alles wat nieuw was in huis",
      "AR": "Niet echt",
      "PA": "Samen met 19 verticaal je ouders",
      "NS": "Brengt ons overal gratis in het weekend",
      "SI": "Ja, geleerd in colombia"
    }
  },
  "celebration": {
    "title": "Yaaaay!!! Anniversary puzzel opgelost!",
    "message": "Deze puzzel bevat allemaal verwijzingen naar onze relatie en ons samen zijn. Inmiddels al 5 jaar!! Echt bizar en bijzonder dat het al zo lang is. Dit jaar ook nog een hele mooie stap gezet samen door het kopen van ons huisje en verhuizen naar Amsterdam. Dat was echt weer even next level.\n\nIk wil je graag laten weten dat ik heel gelukkig ben met je. In de leuke tijden maar in de mindere tijden. We doen heel veel leuke dingen en het is altijd gezellig. Maar je kan er ook heel goed mee om gaan als ik weer eens ongezellig ben of last heb van een van mn kwaaltjes. Ik hoop dat we voor altijd samen blijven en dat ik bij 10 jaar weer een nieuwe puzzel kan maken!! \n\nIn de puzzel zaten ook een aantal hints naar het kado. Ik hoop dat je het leuk vindt!\n\nIk hou van je <3"
  }
}
```

Note: the vertical clue for `"LA"` appears twice in the original (clue 2 = "Enige over van het jaren 20 tafeltje" and clue 43 = "Ciao Bel_"). Since clues are keyed by answer word now, and two vertical words both spell "LA", we need to handle this. Looking at the grid: position (1,0) down gives "LA" (row 1 col 0: blank in row 0 col 0 is 'S' — actually let me re-check). The grid shows row 10 col 10 starts a vertical "LA" too? Actually looking carefully: clue 2 vertical starts at (0,2) going down = L,A → "LA". Clue 43 vertical: the comment says "Ciao Bel_" → "LA" at position (10,11) going down = L,A. Two different "LA" words in the vertical direction. Since word-keyed clues can't have duplicate keys in the same direction, use a position suffix for the second one: key `"LA"` for the first, key `"LA:10,11"` for the second (row,col of word start). The code will need to handle this — try exact word match first, then fall back to `"WORD:row,col"` format.

Update the JSON vertical clues to use this convention:

```json
"LA": "Enige over van het jaren 20 tafeltje",
"LA:10,11": "Ciao Bel_",
```

- [ ] **Step 3: Create puzzles/index.json**

```json
[
  {
    "id": "puzzle-1",
    "title": "5 Jaar",
    "subtitle": "30 mei 2026",
    "description": "Onze eerste kruiswoordpuzzel",
    "file": "puzzle-1.json"
  }
]
```

- [ ] **Step 4: Verify JSON is valid**

Run: `cat puzzles/puzzle-1.json | python3 -m json.tool > /dev/null && echo "Valid JSON" || echo "Invalid JSON"`
Run: `cat puzzles/index.json | python3 -m json.tool > /dev/null && echo "Valid JSON" || echo "Invalid JSON"`

Expected: both print "Valid JSON"

- [ ] **Step 5: Commit**

```bash
git add puzzles/puzzle-1.json puzzles/index.json
git commit -m "feat: extract puzzle-1 data to JSON and create manifest"
```

---

### Task 2: Update buildClues to accept word-keyed clues

Modify `buildClues()` in `index.html` so it can match clues by answer word (reading letters from the solution grid) instead of by number. This is a standalone change — the existing puzzle still works because we'll convert its number-keyed clues to word-keyed format.

**Files:**
- Modify: `index.html` — `buildClues()` function (lines 371–391)

**Consumes:** `parseGrid()` output (solution array), `assignNumbers()` output (numbered words)
**Produces:** `buildClues(numberedWords, clueTexts, solution)` — now accepts solution grid and word-keyed clue texts

- [ ] **Step 1: Modify buildClues to match by answer word**

Replace the `buildClues` function in `index.html` (lines 371–391) with:

```javascript
function buildClues(numberedWords, clueTexts, solution) {
  const clues = { horizontal: [], vertical: [] };
  for (const word of numberedWords) {
    let wordStr = '';
    for (let i = 0; i < word.length; i++) {
      if (word.direction === 'horizontal') {
        wordStr += solution[word.row][word.col + i];
      } else {
        wordStr += solution[word.row + i][word.col];
      }
    }

    const dirClues = clueTexts[word.direction];
    let text = dirClues[wordStr];
    if (!text) {
      const posKey = `${wordStr}:${word.row},${word.col}`;
      text = dirClues[posKey];
    }
    if (!text) {
      console.error(
        `Missing clue: ${word.direction} "${wordStr}" at (${word.row},${word.col}), number ${word.number}`
      );
      continue;
    }

    clues[word.direction].push({
      number: word.number,
      row: word.row,
      col: word.col,
      length: word.length,
      direction: word.direction,
      clue: text,
    });
  }
  return clues;
}
```

This tries an exact word match first (e.g. `"SLOT"`), then falls back to a position-qualified key (e.g. `"LA:10,11"`) for duplicate words in the same direction.

- [ ] **Step 2: Update the buildClues call site**

Replace the puzzle construction (lines 493–558) to use word-keyed clues and pass `solution`:

```javascript
const puzzle = {
  rows: 12,
  cols: 13,
  grid,
  solution,
  clues: buildClues(assignNumbers(detectWords(grid, 12, 13)), {
    horizontal: {
      "SLOT": "Vraag ik altijd over voor we gaan slapen",
      "KIND": "Wil ik heeeel graag",
      "VI": "Samen met 25 verticaal 👶",
      "CAFE": "Tent met taps",
      "KAS": "Restaurant voor mijn 25ste",
      "DAN": "Past voor ken en sen",
      "KAAS": "Favoriet op de borrelplank",
      "MELK": "Ga ik van scheten",
      "EN": "Jij _ ik",
      "KNAP": "Dat ben jij!",
      "ADS": "Computer Science vak",
      "RUMOURS": "Album covers",
      "MES": "Hebben we elkaar allebei een keer kado gedaan",
      "AMSTERDAM": "Waar ons huis woont",
      "OE": "Jouw favoriete stopwoordje van mij",
      "UUT": "Halve marathon en mijn kantoor",
      "FLINKE": "Goede grootte",
      "IN": "Verbindt vriend en tiem",
      "CUE": "Annivesary resto",
      "FLOOR": "Wij zitten op de vierde",
      "AA": "Gaan we zondag nodig hebben",
      "MAM": "__ Bellen",
      "JAPANS": "Land van oorsprong van jouw 20 horizontaal",
      "LAS": "Joe speedboot activiteit in het verleden",
      "ARA": "Standaard gevraagd in een zweedse puzzel",
      "SIERAAD": "Een van mijn eerste kados van Pig en Hen"
    },
    vertical: {
      "SCHERP": "Gewenste eigenschap van 20 horizontaal",
      "LA": "Enige over van het jaren 20 tafeltje",
      "OF": "Een keuze",
      "TE": "Nooit goed",
      "KAAPSE": "Karel die een lekker biertje is",
      "ISS": "Ga je heen als je austronouten droom uitkomt",
      "VAL": "Jij raakte je horloge kwijt en ik een stukje van mn kin tijdens een gezamelijke __",
      "INKT": "Past voor vis en patroon",
      "KAART": "Ons huis ligt west op de __ van amsterdam",
      "DESEM": "Fave brood, isa proof en homemade",
      "KNUS": "Ons huisje is _ en gezellig",
      "MDMA": "Maakt ons happy op feestjes",
      "NU": "Wanneer zijn we 5 jaar samen?",
      "KOMBUCHA": "Vind ik stinken maar wel lekker drinken",
      "MA": "Samen met 40 verticaal je ouders",
      "ROFFA": "Van __ en 22 verticaal kwamen we",
      "DELLA": "Van __ en 22 verticaal kwamen we",
      "LE": "Samen met 7 horizontaal 👶",
      "RIOJA": "Fave rode wijn",
      "TU": "Universiteit waar we inginieur zijn geworden",
      "IO": "\"Makkelijke studie\"",
      "NOMAD": "Wij in Parijs",
      "KRAS": "Zat al snel op alles wat nieuw was in huis",
      "AR": "Niet echt",
      "PA": "Samen met 19 verticaal je ouders",
      "NS": "Brengt ons overal gratis in het weekend",
      "SI": "Ja, geleerd in colombia",
      "LA:10,11": "Ciao Bel_"
    }
  }, solution),
};
```

- [ ] **Step 3: Open in browser and verify the puzzle renders correctly**

Run: `python3 -m http.server 8000` from the project root, then open `http://localhost:8000` in a browser.

Expected: The puzzle renders identically to before — same grid, same clue numbers, same clue texts. All 26 horizontal and 28 vertical clues appear correctly. No errors in the browser console.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: buildClues now matches by answer word instead of number"
```

---

### Task 3: Add hash routing and dual-view DOM structure

Add the selection view HTML, the puzzle view wrapper, and hash routing logic. After this task, the app shows the selection page at `/` and loads puzzle-1 at `#puzzle-1` — but the selection page is a placeholder (just a header, no cards yet).

**Files:**
- Modify: `index.html` — HTML body structure and new routing JS

**Consumes:** Existing puzzle engine functions
**Produces:**
- `route()` — reads hash and shows correct view
- `loadPuzzle(id)` — fetches JSON, parses, renders, restores progress
- `showSelection()` — shows selection view, hides puzzle view
- Selection view DOM element (`#view-selection`)
- Puzzle view DOM element (`#view-puzzle`)

- [ ] **Step 1: Restructure the HTML body**

Replace the current `<body>` content (lines 283–301) with two views. The puzzle view wraps the existing `<header>`, `<main>`, grid, and clues. The selection view is a new sibling:

```html
<body>
  <div id="view-selection" style="display:none">
    <header>
      <h1>Paubella Crux</h1>
    </header>
    <main id="selection-content">
      <p>Puzzels laden...</p>
    </main>
  </div>

  <div id="view-puzzle" style="display:none">
    <header>
      <a id="back-link" href="#">← Puzzels</a>
      <h1 id="puzzle-title">Paubella Crux</h1>
      <p class="subtitle" id="puzzle-subtitle"></p>
    </header>
    <main>
      <div id="grid-container"></div>
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
    </main>
  </div>
```

- [ ] **Step 2: Add CSS for the back link and selection page**

Add these styles after the existing `.btn-reset:hover` rule (line 280):

```css
#back-link {
  display: inline-block;
  margin-bottom: 0.5rem;
  color: #a08060;
  text-decoration: none;
  font-size: 0.9rem;
}

#back-link:hover {
  color: #2c2420;
}

#view-selection header {
  text-align: center;
  margin-bottom: 2rem;
}

#selection-content {
  max-width: 600px;
  width: 100%;
}
```

- [ ] **Step 3: Extract puzzle engine into loadPuzzle function**

Remove the current inline puzzle construction and init code (lines 478–560 and 1009–1011). Replace the entire `<script>` section's init flow with:

Keep all existing functions (`parseGrid`, `detectWords`, `assignNumbers`, `buildClues`, `validatePuzzle`, `buildNumberMap`, `buildWordLookup`, `getCell`, `getWordCells`, `findWordAt`, `firstEmptyCell`, `renderGrid`, `renderClues`, `updateHighlighting`, `selectWord`, `selectClue`, `placeLetter`, `advanceCursor`, `deleteLetter`, `saveProgress`, `restoreProgress`, `checkCompletion`, `showCelebration`, `resetPuzzle`).

But move these to module-level variables instead of being tied to a single puzzle:

```javascript
let puzzle = null;
let state = null;
let wordLookup = null;
let clueElements = {};
let currentPuzzleId = null;
```

Update `saveProgress` and `restoreProgress` to use per-puzzle keys:

```javascript
function saveProgress() {
  if (!currentPuzzleId) return;
  localStorage.setItem(
    `crux-progress-${currentPuzzleId}`,
    JSON.stringify(state.gridLetters)
  );
}

function restoreProgress() {
  if (!currentPuzzleId) return;
  const saved = localStorage.getItem(`crux-progress-${currentPuzzleId}`);
  if (!saved) return;

  let parsed;
  try { parsed = JSON.parse(saved); } catch (e) { return; }

  if (!Array.isArray(parsed) || parsed.length !== puzzle.rows) return;
  if (!parsed.every(row => Array.isArray(row) && row.length === puzzle.cols)) return;

  for (let r = 0; r < puzzle.rows; r++) {
    for (let c = 0; c < puzzle.cols; c++) {
      if (puzzle.grid[r][c] !== null && typeof parsed[r][c] === 'string' && /^[A-Z]$/.test(parsed[r][c])) {
        state.gridLetters[r][c] = parsed[r][c];
        getCell(r, c).querySelector('.cell-letter').textContent = parsed[r][c];
      }
    }
  }

  checkCompletion();
}
```

Update `resetPuzzle` to use per-puzzle key:

```javascript
function resetPuzzle() {
  state.completed = false;
  state.direction = 'horizontal';
  state.cursorCell = null;
  state.selectedWord = null;
  state.lastClickedCell = null;
  for (let r = 0; r < puzzle.rows; r++) {
    for (let c = 0; c < puzzle.cols; c++) {
      if (puzzle.grid[r][c] !== null) {
        state.gridLetters[r][c] = '';
        getCell(r, c).querySelector('.cell-letter').textContent = '';
      }
    }
  }
  updateHighlighting();
  if (currentPuzzleId) {
    localStorage.removeItem(`crux-progress-${currentPuzzleId}`);
    localStorage.removeItem(`crux-completed-${currentPuzzleId}`);
  }
}
```

Update `checkCompletion` to write the completed flag:

```javascript
function checkCompletion() {
  for (let r = 0; r < puzzle.rows; r++) {
    for (let c = 0; c < puzzle.cols; c++) {
      if (puzzle.grid[r][c] !== null && state.gridLetters[r][c] === '') {
        return;
      }
    }
  }

  for (let r = 0; r < puzzle.rows; r++) {
    for (let c = 0; c < puzzle.cols; c++) {
      if (puzzle.grid[r][c] !== null && state.gridLetters[r][c] !== puzzle.solution[r][c]) {
        return;
      }
    }
  }

  state.completed = true;
  if (currentPuzzleId) {
    localStorage.setItem(`crux-completed-${currentPuzzleId}`, 'true');
  }
  setTimeout(() => showCelebration(), 300);
}
```

Update `showCelebration` to use the puzzle's celebration data:

```javascript
function showCelebration() {
  // ... confetti code stays the same ...

  setTimeout(() => {
    const backdrop = document.createElement('div');
    backdrop.classList.add('celebration-backdrop');

    const card = document.createElement('div');
    card.classList.add('celebration-card');

    const header = document.createElement('h2');
    header.classList.add('celebration-header');
    header.textContent = puzzle.celebration
      ? puzzle.celebration.title
      : 'Puzzel opgelost!';

    const message = document.createElement('p');
    message.classList.add('celebration-message');
    message.textContent = puzzle.celebration
      ? puzzle.celebration.message
      : '';

    // ... rest of buttons/card assembly stays the same ...
  }, 500);
}
```

Add the `loadPuzzle` function:

```javascript
async function loadPuzzle(id) {
  const gridContainer = document.getElementById('grid-container');
  gridContainer.innerHTML = '';
  document.getElementById('clues-horizontal').innerHTML = '';
  document.getElementById('clues-vertical').innerHTML = '';
  clueElements = {};

  const response = await fetch(`puzzles/${id}.json`);
  if (!response.ok) {
    console.error(`Failed to load puzzle ${id}: ${response.status}`);
    return;
  }
  const data = await response.json();

  const { grid, solution } = parseGrid(data.grid);
  const rows = data.grid.length;
  const cols = data.grid[0].length;

  const numberedWords = assignNumbers(detectWords(grid, rows, cols));
  const clues = buildClues(numberedWords, data.clues, solution);

  currentPuzzleId = id;
  puzzle = { rows, cols, grid, solution, clues, celebration: data.celebration || null };
  validatePuzzle(puzzle);

  state = {
    direction: 'horizontal',
    cursorCell: null,
    selectedWord: null,
    lastClickedCell: null,
    gridLetters: grid.map(row => row.map(cell => cell === null ? null : '')),
    completed: false,
  };
  wordLookup = buildWordLookup(puzzle);

  document.getElementById('puzzle-title').textContent = data.title || 'Paubella Crux';
  document.getElementById('puzzle-subtitle').textContent = data.subtitle || '';

  renderGrid(puzzle);
  renderClues(puzzle);
  restoreProgress();
}
```

Add routing functions:

```javascript
function showSelection() {
  document.getElementById('view-selection').style.display = '';
  document.getElementById('view-puzzle').style.display = 'none';
}

function showPuzzleView() {
  document.getElementById('view-selection').style.display = 'none';
  document.getElementById('view-puzzle').style.display = '';
}

async function route() {
  const hash = location.hash.slice(1);
  if (hash) {
    await loadPuzzle(hash);
    showPuzzleView();
  } else {
    showSelection();
  }
}

window.addEventListener('hashchange', route);
route();
```

- [ ] **Step 4: Move event listeners to be puzzle-aware**

The clue click, grid click, and keyboard listeners (lines 1013–1098) should guard against `puzzle` being null and stay attached permanently. They already check `state.completed`, just add a null guard:

```javascript
document.getElementById('clues-container').addEventListener('click', (e) => {
  if (!puzzle || state.completed) return;
  const item = e.target.closest('.clue-item');
  if (!item) return;
  const direction = item.dataset.direction;
  const number = parseInt(item.dataset.number);
  const clue = puzzle.clues[direction].find(c => c.number === number);
  if (clue) selectClue(clue);
});

document.getElementById('grid-container').addEventListener('click', (e) => {
  if (!puzzle || state.completed) return;
  const cell = e.target.closest('.cell-white');
  if (!cell) return;
  selectWord(parseInt(cell.dataset.row), parseInt(cell.dataset.col));
});

document.addEventListener('keydown', (e) => {
  if (!puzzle || state.completed) return;
  // ... rest stays the same ...
});
```

- [ ] **Step 5: Open in browser and verify routing works**

Run: `python3 -m http.server 8000`

Test 1: Open `http://localhost:8000` — should see the selection page header ("Paubella Crux") and "Puzzels laden..." text.

Test 2: Navigate to `http://localhost:8000/#puzzle-1` — should see the full puzzle with grid, clues, working input. Title should show "5 Jaar", subtitle "30 mei 2026".

Test 3: Click the "← Puzzels" back link — should return to the selection page.

Test 4: Press browser back button from puzzle view — should return to selection page.

Test 5: Type letters, refresh page, navigate to `#puzzle-1` again — progress should be saved and restored.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add hash routing with selection/puzzle dual-view structure"
```

---

### Task 4: Build the selection page with puzzle cards

Fetch `puzzles/index.json` and render puzzle cards with title, subtitle, description, progress indicator, and completed checkmark.

**Files:**
- Modify: `index.html` — `showSelection()` function, new CSS for cards

**Consumes:** `puzzles/index.json`, localStorage keys `crux-progress-{id}` and `crux-completed-{id}`
**Produces:** Fully functional selection page with cards that link to puzzles

- [ ] **Step 1: Add CSS for puzzle cards**

Add after the `#selection-content` styles:

```css
.puzzle-cards {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.puzzle-card {
  background: #fffdf8;
  border: 2px solid #e8ddd0;
  border-radius: 8px;
  padding: 1.25rem 1.5rem;
  cursor: pointer;
  transition: border-color 200ms, box-shadow 200ms;
}

.puzzle-card:hover {
  border-color: #d4b896;
  box-shadow: 0 2px 8px rgba(44, 36, 32, 0.08);
}

.puzzle-card-header {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  margin-bottom: 0.25rem;
}

.puzzle-card-title {
  font-family: Georgia, serif;
  font-size: 1.2rem;
  font-weight: 700;
  color: #2c2420;
}

.puzzle-card-check {
  color: #98D8AA;
  font-size: 1.2rem;
}

.puzzle-card-subtitle {
  font-size: 0.8rem;
  color: #a08060;
  margin-bottom: 0.5rem;
}

.puzzle-card-description {
  font-size: 0.9rem;
  color: #4a3f35;
  margin-bottom: 0.75rem;
}

.puzzle-card-progress {
  font-size: 0.8rem;
  color: #a08060;
}

.progress-bar {
  height: 4px;
  background: #e8ddd0;
  border-radius: 2px;
  margin-top: 0.35rem;
  overflow: hidden;
}

.progress-bar-fill {
  height: 100%;
  background: #d4b896;
  border-radius: 2px;
  transition: width 300ms;
}
```

- [ ] **Step 2: Add helper to compute progress from localStorage**

```javascript
function getPuzzleProgress(id, gridRows) {
  const completed = localStorage.getItem(`crux-completed-${id}`);
  if (completed === 'true') return { filled: -1, total: -1, completed: true };

  const saved = localStorage.getItem(`crux-progress-${id}`);
  if (!saved) return { filled: 0, total: 0, completed: false };

  let parsed;
  try { parsed = JSON.parse(saved); } catch (e) { return { filled: 0, total: 0, completed: false }; }

  let filled = 0;
  let total = 0;
  for (const row of parsed) {
    if (!Array.isArray(row)) continue;
    for (const cell of row) {
      if (cell !== null) {
        total++;
        if (typeof cell === 'string' && /^[A-Z]$/.test(cell)) filled++;
      }
    }
  }

  return { filled, total, completed: false };
}
```

- [ ] **Step 3: Update showSelection to fetch manifest and render cards**

```javascript
async function showSelection() {
  document.getElementById('view-selection').style.display = '';
  document.getElementById('view-puzzle').style.display = 'none';

  const content = document.getElementById('selection-content');

  try {
    const response = await fetch('puzzles/index.json');
    const manifest = await response.json();

    content.innerHTML = '';
    const cards = document.createElement('div');
    cards.classList.add('puzzle-cards');

    for (const entry of manifest) {
      const card = document.createElement('div');
      card.classList.add('puzzle-card');
      card.addEventListener('click', () => {
        location.hash = entry.id;
      });

      const progress = getPuzzleProgress(entry.id);

      const headerDiv = document.createElement('div');
      headerDiv.classList.add('puzzle-card-header');

      const title = document.createElement('span');
      title.classList.add('puzzle-card-title');
      title.textContent = entry.title;
      headerDiv.appendChild(title);

      if (progress.completed) {
        const check = document.createElement('span');
        check.classList.add('puzzle-card-check');
        check.textContent = '✓';
        headerDiv.appendChild(check);
      }

      card.appendChild(headerDiv);

      const subtitle = document.createElement('div');
      subtitle.classList.add('puzzle-card-subtitle');
      subtitle.textContent = entry.subtitle;
      card.appendChild(subtitle);

      const desc = document.createElement('div');
      desc.classList.add('puzzle-card-description');
      desc.textContent = entry.description;
      card.appendChild(desc);

      if (!progress.completed && progress.total > 0) {
        const progressDiv = document.createElement('div');
        progressDiv.classList.add('puzzle-card-progress');
        progressDiv.textContent = `${progress.filled} / ${progress.total} letters`;

        const bar = document.createElement('div');
        bar.classList.add('progress-bar');
        const fill = document.createElement('div');
        fill.classList.add('progress-bar-fill');
        fill.style.width = `${Math.round((progress.filled / progress.total) * 100)}%`;
        bar.appendChild(fill);
        progressDiv.appendChild(bar);
        card.appendChild(progressDiv);
      }

      cards.appendChild(card);
    }

    content.appendChild(cards);
  } catch (e) {
    content.innerHTML = '<p>Kon puzzels niet laden.</p>';
    console.error('Failed to load manifest:', e);
  }
}
```

- [ ] **Step 4: Open in browser and verify the selection page**

Run: `python3 -m http.server 8000`

Test 1: Open `http://localhost:8000` — should see the "Paubella Crux" header and one puzzle card for "5 Jaar" with subtitle "30 mei 2026" and description.

Test 2: Click the card — should navigate to `#puzzle-1` and load the puzzle.

Test 3: Type some letters in the puzzle, then click "← Puzzels" — card should now show a progress bar (e.g. "3 / 62 letters").

Test 4: Complete the puzzle (or manually set `localStorage.setItem('crux-completed-puzzle-1', 'true')`), go back to selection — card should show green checkmark, no progress bar.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add selection page with puzzle cards, progress, and completion state"
```

---

### Task 5: Verify end-to-end and clean up

Final verification that everything works together. Remove any leftover hardcoded puzzle data from `index.html` if it still exists.

**Files:**
- Modify: `index.html` — remove any leftover inline puzzle data
- Verify: `puzzles/puzzle-1.json`, `puzzles/index.json`

- [ ] **Step 1: Verify no hardcoded puzzle data remains in index.html**

Search `index.html` for the old inline grid data. If lines like `const { grid, solution } = parseGrid([` still exist outside of `loadPuzzle`, remove them. The only puzzle data should come from fetched JSON.

- [ ] **Step 2: Full end-to-end test**

Run: `python3 -m http.server 8000`

Test flow:
1. Open `http://localhost:8000` — selection page shows
2. Click puzzle-1 card — puzzle loads, grid renders, clues show
3. Type letters — they appear, cursor advances
4. Click a clue — word highlights, cursor moves
5. Arrow keys — navigate between words
6. Backspace — deletes letters
7. Refresh page — progress is preserved
8. Click "← Puzzels" — back to selection, progress bar shows
9. Navigate to `#puzzle-1` via URL — puzzle loads directly
10. Browser back — returns to selection

No console errors throughout.

- [ ] **Step 3: Commit final state**

```bash
git add index.html
git commit -m "chore: remove leftover inline puzzle data, verify clean state"
```

(Skip this commit if there were no changes to make.)
