# ACRONYM.QUIZ — Game Idea & Spec

Terminal/CLI aesthetic (dark green-on-black, JetBrains Mono, scanlines, blinking cursor) matching the design proposal in `index.html`.

---

## UI / Language

- Language toggle: **DE | EN** (player picks at start; all UI labels, category names, and acronyms switch language where translations exist; acronyms themselves are language-neutral)
- Entire UI follows the terminal/CLI look from the design proposal:
  - Dark background `#08090b`, green accents `#4ade80`
  - JetBrains Mono font
  - Terminal window chrome (three dots, `user@quizbox:~$` prompt)
  - Optional scanline overlay
  - Blinking cursor on titles

---

## Acronym Database ("DB")

The game is backed by a local acronym database (JSON/JS object bundled in the page).

### Schema

| Field | Type | Example | Notes |
|-------|------|---------|-------|
| `acronym` | string | `LSTM` | The acronym itself |
| `full` | string | `Long Short-Term Memory` | Correct expansion |
| `cat` | string | `IT` | Top-level category |
| `subcat` | string | `AI` | Sub-category within the category |
| `source` | string | `wikipedia` | Where the definition was sourced |
| `recursive` | boolean | `false` | `true` for recursive acronyms (e.g. GNU) |
| `link` | string (URL) | `https://en.wikipedia.org/wiki/Long_short-term_memory` | Wikipedia or explanation-site link |

### Content scope

Lots of acronyms across IT and adjacent fields. Examples by category/sub-category:

**IT → AI**
- LSTM — Long Short-Term Memory
- xLSTM — Extended Long Short-Term Memory
- RNN — Recurrent Neural Network
- GPT — Generative Pre-trained Transformer
- NLP — Natural Language Processing
- CNN — Convolutional Neural Network
- ...

**IT → Networking**
- TCP — Transmission Control Protocol
- BGP — Border Gateway Protocol
- ISDN — Integrated Services Digital Network
- DNS — Domain Name System
- HTTP — HyperText Transfer Protocol
- ...

**IT → Hardware**
- SSD — Solid State Drive
- CPU — Central Processing Unit
- GPU — Graphics Processing Unit
- ...

**Software Development / SW Architecture**
- SDLC — Software Development Life Cycle
- SOLID — (S)ingle Responsibility, (O)pen/Closed, (L)iskov Substitution, (I)nterface Segregation, (D)ependency Inversion
- CI/CD — Continuous Integration / Continuous Deployment
- DRY — Don't Repeat Yourself
- KISS — Keep It Simple, Stupid
- ...

**Security**
- CSRF — Cross-Site Request Forgery
- XSS — Cross-Site Scripting
- ...

**Cloud**
- IaaS — Infrastructure as a Service
- PaaS — Platform as a Service
- SaaS — Software as a Service
- ...

(More categories and acronyms to be added — the DB should be large.)

### DB Browser

Separate screen accessible from the start menu. Player can:
- Browse all acronyms (grouped by category → sub-category)
- Search/filter by text (acronym or full expansion)
- Filter by category, sub-category, recursive flag
- Click an entry to see the full row + external link
- Switch language (DE/EN) for labels where available

---

## Categories & Sub-Categories

Acronyms are grouped hierarchically:

```
Category (e.g. IT)
  └── Sub-Category (e.g. AI, Networking, Hardware, Security, ...)
        └── Acronym (e.g. LSTM, TCP, SSD, ...)
```

At quiz start the player selects which categories/sub-categories to include (checkboxes in the terminal style from the proposal). Default: all enabled.

---

## Quiz Modes

### 1. Multiple Choice (from the proposal)

- Show acronym, present 4 options (A–D), player picks via keyboard or click
- Streak counter, score, timer bar
- 10 questions per round
- Results screen with run.log recap (✔/✘ per question, score breakdown)

### 2. Glücksrad (Wheel-of-Fortune style)

Like the classic "Glücksrad" / "Wheel of Fortune" letter game:

- The full expansion is shown as blanks: `_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _`
- Player picks letters one at a time:
  - Click a letter button (A–Z grid below the puzzle)
  - OR type a letter on the keyboard
- Correct letters fill in all matching positions
- Wrong letters cost points (or lives / time)
- At any point the player can choose to **Solve**: type the complete remaining text
  - If correct → full points bonus
  - If wrong → penalty
- Keyboard or on-screen letter buttons — both work
- Terminal aesthetic: letters rendered in monospace, blanks as underscores, revealed letters in green `#4ade80`

### 3. Recursive Acronyms (special section)

A dedicated section for recursive acronyms (acronyms where the first letter stands for the acronym itself):

- **Info screen:** explains what recursive acronyms are
  - "A recursive acronym is an acronym that refers to itself. The first letter (or word) of the expansion is the acronym itself."
  - Example: **GNU** = "GNU's Not Unix" — the G stands for GNU itself
- **List view:** shows all known recursive acronyms as facts (not a quiz, just a browsable list):
  - GNU — GNU's Not Unix
  - WINE — Wine Is Not an Emulator
  - BING — Bing Is Not Google
  - YAML — YAML Ain't Markup Language
  - PHP — PHP: Hypertext Preprocessor
  - PIP — Pip Installs Packages
  - Hurd — Hird of Unix-Replacing Daemons (where Hird = Hurd of Interfaces Representing Depth)
  - ... (more to be added)
- **Quiz mode:** Glücksrad-style — reveal the expansion letter by letter, then solve

---

## Difficulty Levels

From the proposal:
- **junior** — common acronyms (TCP, API, URL)
- **senior** — mid-tier (BGP, SDLC, CSRF)
- **greybeard** — obscure / deep-cut (ISDN, xLSTM, Hurd)

Difficulty filters which acronyms from the DB are eligible for the quiz.

---

## Game Flow

```
START SCREEN
  ├── Select language: DE | EN
  ├── Select topics (categories/sub-categories)
  ├── Select difficulty: junior | senior | greybeard
  ├── Choose mode: Multiple Choice | Glücksrad | Recursive Acronyms
  └── ▶ RUN QUIZ

QUIZ SCREEN (mode-dependent)
  ├── Multiple Choice → A–D options, timer, streak, score
  ├── Glücksrad → letter reveal, solve input
  └── Recursive → info → list → Glücksrad quiz

RESULTS SCREEN
  ├── Score, rank, streak
  ├── run.log recap per acronym
  └── ↻ RUN AGAIN | share result
```

---

## Technical Notes

- Single-file `index.html` (React + Babel standalone, like the proposal bundle)
- Acronym DB embedded as a JS object in the page (no backend)
- All client-side, served via GitHub Pages at `https://github.freaxnx01.ch/game-acronym-quiz/`
- Keyboard + mouse input throughout
- LocalStorage for high score persistence
