# "Then vs. now" mode — design

**Status:** approved in chat 2026-10-31, spec awaiting review
**Scope:** `index.html` only (data + one new mode). No new files at runtime.

## Goal

Some acronyms changed their meaning over time (PHP was *Personal Home Page*, RAID's
*I* was *Inexpensive*). A separate game mode asks for the **original** meaning, with
today's meaning planted among the answers as a trap, then reveals *then → now*.

Decisions from the brainstorming session:

- Own mode in the mode picker (not a 5th entry in the Acronym Types hub).
- Multiple choice, always asks for the original meaning.
- IT acronyms only (no companies/organisations such as WWF, KFC, UBS).
- After answering, the reveal stays until ENTER / click — no auto-advance.

## Data

New optional fields on `ACRONYMS` entries:

| Field | Type | Meaning |
|-------|------|---------|
| `then` | string | Original expansion. Presence puts the entry in the mode pool. |
| `thenLang` | `"de"`/`"en"` | Only when `then` is in a different language than `full` (ABAP). |
| `orphan` | boolean | Today it is officially no longer an acronym. `full` then keeps the original expansion so the other modes still work; `then` equals `full`. |

Pool: 21 entries. Every expansion below is verified against its Wikipedia article
before it is committed; wording follows Wikipedia where it differs.

**Existing entries — add `then` only:**

| Acronym | `then` | `full` (unchanged) |
|---------|--------|--------------------|
| PHP | Personal Home Page | PHP: Hypertext Preprocessor |
| YAML | Yet Another Markup Language | YAML Ain't Markup Language |
| WINE | WINdows Emulator | Wine Is Not an Emulator |
| RPM | Red Hat Package Manager | RPM Package Manager |
| XBMC | Xbox Media Center | XBMC Media Center |

**New entries** (`source:"wikipedia"`, `recursive:false`, Wikipedia `link`):

| Acronym | `then` | `full` | cat · subcat | difficulty |
|---------|--------|--------|--------------|------------|
| ABAP | Allgemeiner Berichts-Aufbereitungs-Prozessor | Advanced Business Application Programming | SW Dev · Tools | senior |
| SAP | Systemanalyse Programmentwicklung | Systeme, Anwendungen und Produkte in der Datenverarbeitung | IT · Cloud | junior |
| RAID | Redundant Array of Inexpensive Disks | Redundant Array of Independent Disks | IT · Hardware | senior |
| DVD | Digital Video Disc | Digital Versatile Disc | IT · Hardware | junior |
| RSS | RDF Site Summary | Really Simple Syndication | IT · Web | senior |
| SQL | Structured English Query Language | Structured Query Language | SW Dev · Tools | junior |
| GCC | GNU C Compiler | GNU Compiler Collection | SW Dev · Tools | senior |
| GIMP | General Image Manipulation Program | GNU Image Manipulation Program | SW Dev · Tools | senior |
| KDE | Kool Desktop Environment | K Desktop Environment | SW Dev · Tools | senior |
| ARM | Acorn RISC Machine | Advanced RISC Machines | IT · Hardware | senior |
| GSM | Groupe Spécial Mobile | Global System for Mobile Communications | IT · Networking | senior |
| LLVM | Low Level Virtual Machine | (orphan) | SW Dev · Tools | greybeard |
| SOAP | Simple Object Access Protocol | (orphan) | IT · Web | greybeard |
| XFCE | XForms Common Environment | (orphan) | SW Dev · Tools | greybeard |
| CUPS | Common UNIX Printing System | (orphan) | SW Dev · Tools | greybeard |
| VLC | VideoLAN Client | (orphan) | SW Dev · Tools | senior |

Language of decoys: `makeDistractors()` needs the language of the text it mutates
(see `acronymLang()`). SAP is German in both `then` and `full` → `lang:"de"`.
ABAP's `then` is German but its `full` is English → no `lang`, plus a third optional
field `thenLang:"de"`, used only when building this mode's decoys
(`a.thenLang||acronymLang(a)`). Normal MC keeps using `acronymLang(a)` on `full`.

SQL's `then` is the expansion of its predecessor name SEQUEL, so its letters do not
spell SQL. That is historically correct and kept; generated decoys follow `then`.

Each new acronym is added to exactly one `DIFFICULTY` list (column above), so the
normal modes can pick it.

## Gameplay

- **Mode picker:** new button `state.mode==='thenNow'`, label
  `thenVsNow` — EN "Then vs. Now", DE "Früher vs. heute".
- **Pool:** `ACRONYMS.filter(a=>a.then)`. Topic and difficulty filters are ignored
  (the pool is too small to filter). 10 questions per round.
- **Question line:** EN "what did it originally stand for?", DE "Wofür stand es
  ursprünglich?" — replaces the "expand the acronym" line.
- **Answers (4):**
  1. `then` — correct.
  2. `full` — the trap, unless `orphan`.
  3. Remaining slots from `makeDistractors(then, …)`; existing fallback applies.
  Options are de-duplicated and shuffled.
- **Scoring, streak, high score, results screen:** same as multiple choice. The run
  log shows `then` as the correct expansion.
- **Reveal:** after answering, the usual correct/wrong highlight plus a line
  `then: <then> → now: <full>` (orphan: `now: no longer an acronym` /
  `heute: kein Akronym mehr`). The question stays until ENTER or a click on a
  `next →` / `weiter →` button. ESC still quits.

## Code shape

Reuse the multiple-choice flow; do not add a parallel renderer.

- `startQuiz()` branches on `thenNow`, builds questions from the `then` pool and sets
  `state.quizData.thenNow=true`.
- Question building: a small `buildThenNowQuestions(pool,n)` next to
  `buildMCQuestions` (correct answer `then`, trap `full`), returning the same question
  shape plus `now`/`orphan` for the reveal.
- `renderMCQuiz` / `mcAnswer` / `mcKeyHandler`: when `quizData.thenNow`, swap the
  question line, render the reveal line + next button after answering, and skip the
  1.5 s auto-advance. Normal MC behaviour is unchanged.

## Testing

Headless Playwright run (scratchpad script, not committed):

- Play a full `thenNow` round: every question's correct option equals the entry's
  `then`; the trap (`full`) is present for non-orphans; 4 distinct options.
- After an answer the reveal line is shown and the question does not advance until
  ENTER.
- Normal MC still auto-advances after 1.5 s.
- Every `ACRONYMS` entry is in some `DIFFICULTY` list; no page errors.

## Out of scope

- Organisation/company acronyms (WWF, UNICEF, CERN, UBS, KFC, AT&T, 3M, SAT, AARP).
- Multi-step histories (RSS's middle stage *Rich Site Summary* is not modelled).
- DB browser changes for `then`.
