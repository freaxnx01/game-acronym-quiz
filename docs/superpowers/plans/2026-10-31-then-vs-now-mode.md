# Then vs. Now Mode Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a "Then vs. Now" / "Früher vs. heute" multiple-choice mode that asks for the original meaning of 20 IT acronyms whose meaning changed.

**Architecture:** Single-file browser game (`index.html`, inline JS, no build). Data gets optional `then` / `thenLang` / `orphan` fields on `ACRONYMS` entries. The mode reuses the existing multiple-choice flow (`renderMCQuiz`, `mcAnswer`, `renderResults`) with its own question builder and a reveal-then-ENTER step instead of the 1.5 s auto-advance.

**Tech Stack:** Vanilla JS in `index.html`; verification with Python Playwright (installed: `python3 -c "import playwright"` works) against `file://…/index.html`.

**Spec:** `docs/superpowers/specs/2026-10-31-then-vs-now-mode-design.md`

## Global Constraints

- Only `index.html` changes at runtime. No new runtime files, no dependencies.
- Pool: IT acronyms only, exactly 20 entries with `then` (4 existing + 16 new).
- Mode key `thenNow`; labels EN "Then vs. Now", DE "Früher vs. heute".
- 10 questions per round; topic and difficulty filters are ignored by this mode.
- Normal multiple choice keeps its 1.5 s auto-advance unchanged.
- Every `ACRONYMS` entry must be in at least one `DIFFICULTY` list.
- Expansions must match Wikipedia wording.
- Test scripts live in the scratchpad, not in the repo:
  `SCRATCH=/tmp/claude-1000/-home-freax-repos-github-freaxnx01-public-freaxnx01-github-io--worktrees-unami/e8fa3fa7-8d23-4e2b-922e-8b76d61ae6fb/scratchpad`
- Commits: Conventional Commits, pushed directly to `main` (repo convention for this game), each ending with the session attribution lines:
  ```
  Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_019LyVMgnVUboS6y7XftNUhL
  ```

## Review Focus

- Orphan entries (LLVM, SOAP, XFCE, CUPS, VLC): `then === full`, so the trap must not be added twice and options must still be 4 distinct strings.
- Short `then` texts with little same-initial vocabulary (KDE, GCC, VLC): `makeDistractors` may return fewer than needed; the fallback must still fill 4 distinct options.
- Pressing ENTER twice quickly after an answer must advance exactly one question (no skipped question).
- ESC during the reveal must quit to start, as everywhere else.
- Switching back to normal MC after a `thenNow` round must auto-advance again (no leftover `thenNow` flag).

Tests for all five are in Task 2's script.

---

### Task 1: Data — `then` fields and 16 new acronyms

**Files:**
- Modify: `index.html` — `ACRONYMS` entries PHP/YAML/RPM/XBMC (around lines 177–184), new block after the XBMC line, `DIFFICULTY` lists (around line 368).
- Test: `$SCRATCH/thennow_data.py` (not committed)

**Interfaces:**
- Produces: `ACRONYMS` entries with `then: string`, optional `thenLang: "de"|"en"`, optional `orphan: true`. `ACRONYMS.filter(a=>a.then).length === 20`.

- [ ] **Step 1: Write the failing data test**

Create `$SCRATCH/thennow_data.py`:

```python
import sys
from playwright.sync_api import sync_playwright
NEW = "ABAP SAP RAID DVD RSS SQL GCC GIMP KDE ARM GSM LLVM SOAP XFCE CUPS VLC".split()
OLD = "PHP YAML RPM XBMC".split()
ORPH = "LLVM SOAP XFCE CUPS VLC".split()
with sync_playwright() as p:
    b = p.chromium.launch(); pg = b.new_page(); errs = []
    pg.on("pageerror", lambda e: errs.append(str(e)))
    pg.goto("file://" + sys.argv[1] + "/index.html"); pg.wait_for_timeout(500)
    acr = pg.evaluate("ACRONYMS")
    diff = pg.evaluate("[...DIFFICULTY.junior,...DIFFICULTY.senior,...DIFFICULTY.greybeard]")
    pool = [a for a in acr if a.get("then")]
    assert len(pool) == 20, f"pool {len(pool)}"
    assert sorted(a["acronym"] for a in pool) == sorted(NEW + OLD)
    for n in NEW:
        assert sum(a["acronym"] == n for a in acr) == 1, f"{n} count"
    for a in pool:
        assert a["link"].startswith("https://en.wikipedia.org/wiki/"), a["acronym"]
        if a["acronym"] in ORPH:
            assert a.get("orphan") is True and a["then"] == a["full"], a["acronym"]
        else:
            assert not a.get("orphan") and a["then"] != a["full"], a["acronym"]
    by = {a["acronym"]: a for a in pool}
    assert by["ABAP"].get("thenLang") == "de" and not by["ABAP"].get("lang")
    assert by["SAP"].get("lang") == "de"
    missing = [a["acronym"] for a in acr if a["acronym"] not in diff]
    assert not missing, f"no difficulty: {missing}"
    assert not errs, errs
    print("DATA OK"); b.close()
```

- [ ] **Step 2: Run it, verify it fails**

Run: `cd ~/repos/github/freaxnx01/public/game-acronym-quiz && timeout 60 python3 $SCRATCH/thennow_data.py "$PWD"`
Expected: `AssertionError: pool 0`

- [ ] **Step 3: Verify expansions against Wikipedia**

Fetch each article below (WebFetch) and confirm the original and current expansion. Use Wikipedia's wording where it differs from this plan's table in Step 4. If an article **contradicts** a `then` value (not just wording), stop and report it instead of guessing.

ABAP, SAP, RAID, DVD, RSS, SQL, GNU_Compiler_Collection, GIMP, KDE, ARM_architecture_family, GSM, LLVM, SOAP, Xfce, CUPS, VLC_media_player, PHP, YAML, Wine_(software), RPM_Package_Manager, Kodi_(software) — all under `https://en.wikipedia.org/wiki/`.

- [ ] **Step 4: Add `then` to the 4 existing entries**

In each of these lines, insert the field right after `full:"…",`:

```
PHP:  then:"Personal Home Page",
YAML: then:"Yet Another Markup Language",
RPM:  then:"Red Hat Package Manager",
XBMC: then:"Xbox Media Center",
```

e.g. `{acronym:"PHP",full:"PHP: Hypertext Preprocessor",then:"Personal Home Page",cat:"SW Dev",…`

- [ ] **Step 5: Add the 16 new entries**

Insert directly after the `{acronym:"XBMC",…},` line:

```js
  // Meaning changed over time ("Then vs. Now" mode: then = original expansion)
  {acronym:"ABAP",full:"Advanced Business Application Programming",then:"Allgemeiner Berichts-Aufbereitungs-Prozessor",thenLang:"de",cat:"SW Dev",subcat:"Tools",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/ABAP"},
  {acronym:"SAP",full:"Systeme, Anwendungen und Produkte in der Datenverarbeitung",then:"Systemanalyse Programmentwicklung",lang:"de",cat:"IT",subcat:"Cloud",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/SAP"},
  {acronym:"RAID",full:"Redundant Array of Independent Disks",then:"Redundant Array of Inexpensive Disks",cat:"IT",subcat:"Hardware",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/RAID"},
  {acronym:"DVD",full:"Digital Versatile Disc",then:"Digital Video Disc",cat:"IT",subcat:"Hardware",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/DVD"},
  {acronym:"RSS",full:"Really Simple Syndication",then:"RDF Site Summary",cat:"IT",subcat:"Web",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/RSS"},
  {acronym:"SQL",full:"Structured Query Language",then:"Structured English Query Language",cat:"SW Dev",subcat:"Tools",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/SQL"},
  {acronym:"GCC",full:"GNU Compiler Collection",then:"GNU C Compiler",cat:"SW Dev",subcat:"Tools",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/GNU_Compiler_Collection"},
  {acronym:"GIMP",full:"GNU Image Manipulation Program",then:"General Image Manipulation Program",cat:"SW Dev",subcat:"Tools",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/GIMP"},
  {acronym:"KDE",full:"K Desktop Environment",then:"Kool Desktop Environment",cat:"SW Dev",subcat:"Tools",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/KDE"},
  {acronym:"ARM",full:"Advanced RISC Machines",then:"Acorn RISC Machine",cat:"IT",subcat:"Hardware",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/ARM_architecture_family"},
  {acronym:"GSM",full:"Global System for Mobile Communications",then:"Groupe Spécial Mobile",cat:"IT",subcat:"Networking",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/GSM"},
  {acronym:"LLVM",full:"Low Level Virtual Machine",then:"Low Level Virtual Machine",orphan:true,cat:"SW Dev",subcat:"Tools",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/LLVM"},
  {acronym:"SOAP",full:"Simple Object Access Protocol",then:"Simple Object Access Protocol",orphan:true,cat:"IT",subcat:"Web",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/SOAP"},
  {acronym:"XFCE",full:"XForms Common Environment",then:"XForms Common Environment",orphan:true,cat:"SW Dev",subcat:"Tools",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/Xfce"},
  {acronym:"CUPS",full:"Common UNIX Printing System",then:"Common UNIX Printing System",orphan:true,cat:"SW Dev",subcat:"Tools",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/CUPS"},
  {acronym:"VLC",full:"VideoLAN Client",then:"VideoLAN Client",orphan:true,cat:"SW Dev",subcat:"Tools",source:"wikipedia",recursive:false,link:"https://en.wikipedia.org/wiki/VLC_media_player"},
```

- [ ] **Step 6: Add the new acronyms to `DIFFICULTY`**

Prepend to the start of each list (the line begins `  junior:[`, `  senior:[`, `  greybeard:[`):

```
junior:    "SAP","DVD","SQL",
senior:    "ABAP","RAID","RSS","GCC","GIMP","KDE","ARM","GSM","VLC",
greybeard: "LLVM","SOAP","XFCE","CUPS",
```

e.g. `  junior:["SAP","DVD","SQL","GPT",…`

- [ ] **Step 7: Run the data test, verify it passes**

Run: `timeout 60 python3 $SCRATCH/thennow_data.py "$PWD"`
Expected: `DATA OK`

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat(data): add original meanings (then) and 16 IT acronyms whose meaning changed

<attribution lines>"
git push
```

---

### Task 2: "Then vs. Now" mode

**Files:**
- Modify: `index.html` — `I18N` en/de (around lines 334–335), mode buttons in `renderStart` (around line 485), `startQuiz` (around line 505), new `buildThenNowQuestions` after `buildMCQuestions` (around line 606), `renderMCQuiz` (around line 607), `mcAnswer` (around line 647).
- Test: `$SCRATCH/thennow_mode.py` (not committed)

**Interfaces:**
- Consumes: `ACRONYMS` entries with `then` / `thenLang` / `orphan` (Task 1); existing `pickN`, `shuffle`, `acronymLang`, `makeDistractors(real,count,lang)`, `quitToStart()`, `t()`.
- Produces: `buildThenNowQuestions(pool:Array, n:number) → Array<{acronym, full /* = then */, now, orphan:boolean, cat, subcat, link, options:string[4], correctIdx:number}>`; `thenNowNext()`; `state.quizData.thenNow === true` for this mode only.

- [ ] **Step 1: Write the failing mode test**

Create `$SCRATCH/thennow_mode.py`:

```python
import sys
from playwright.sync_api import sync_playwright
with sync_playwright() as p:
    b = p.chromium.launch(); pg = b.new_page(); errs = []
    pg.on("pageerror", lambda e: errs.append(str(e)))
    pg.goto("file://" + sys.argv[1] + "/index.html"); pg.wait_for_timeout(500)
    pool = {a["acronym"]: a for a in pg.evaluate("ACRONYMS.filter(a=>a.then)")}

    # question building: 50 rounds to hit orphans and short texts
    for _ in range(50):
        for q in pg.evaluate("buildThenNowQuestions(ACRONYMS.filter(a=>a.then),10)"):
            a = pool[q["acronym"]]
            assert len(q["options"]) == 4 and len(set(q["options"])) == 4, q
            assert q["options"][q["correctIdx"]] == a["then"], q
            if not a.get("orphan"):
                assert a["full"] in q["options"], q
            if a.get("orphan"):
                others = {x["then"] for k, x in pool.items() if k != q["acronym"]}
                assert any(o != a["then"] and o in others for o in q["options"]), q

    # mode button + German question line
    pg.evaluate("setLang('de')")
    assert "Früher vs. heute" in pg.content()
    pg.evaluate("setMode('thenNow');startQuiz()")
    assert pg.evaluate("state.quizData.thenNow") is True
    assert "ursprünglich" in pg.content()
    assert pg.evaluate("state.quizData.questions.length") == 10

    # answer: reveal shown, no auto-advance
    pg.keyboard.press("A"); pg.wait_for_timeout(2500)
    assert pg.evaluate("state.quizData.idx") == 0
    rev = pg.locator("#reveal").inner_text()
    assert "früher" in rev and "heute" in rev, rev

    # double ENTER advances exactly one question
    pg.keyboard.press("Enter"); pg.keyboard.press("Enter"); pg.wait_for_timeout(200)
    assert pg.evaluate("state.quizData.idx") == 1

    # finish the round; run log shows `then`
    for _ in range(9):
        pg.keyboard.press("B"); pg.wait_for_timeout(50)
        pg.locator("#reveal button").click(); pg.wait_for_timeout(50)
    ans = pg.evaluate("state.quizData.answers")
    assert len(ans) == 10
    assert all(x["full"] == pool[x["acronym"]]["then"] for x in ans)
    assert "Prozess beendet" in pg.content()

    # ESC during reveal quits
    pg.evaluate("setMode('thenNow');startQuiz()")
    pg.keyboard.press("A"); pg.keyboard.press("Escape"); pg.wait_for_timeout(100)
    assert pg.evaluate("state.screen") == "start"

    # normal MC still auto-advances, no leftover flag
    pg.evaluate("setMode('mc');startQuiz()")
    assert not pg.evaluate("state.quizData.thenNow")
    pg.keyboard.press("A"); pg.wait_for_timeout(2000)
    assert pg.evaluate("state.quizData.idx") == 1

    assert not errs, errs
    print("MODE OK"); b.close()
```

- [ ] **Step 2: Run it, verify it fails**

Run: `timeout 90 python3 $SCRATCH/thennow_mode.py "$PWD"`
Expected: FAIL with `buildThenNowQuestions is not defined`

- [ ] **Step 3: Add the i18n strings**

In `I18N.en`, replace `suggestAcronym:"Suggest an acronym"}` with:

```js
suggestAcronym:"Suggest an acronym",thenVsNow:"Then vs. Now",thenNowQuestion:"what did it originally stand for?",thenLabel:"then",nowLabel:"now",noLongerAcronym:"no longer an acronym",next:"next →"}
```

In `I18N.de`, replace `suggestAcronym:"Akronym vorschlagen"}` with:

```js
suggestAcronym:"Akronym vorschlagen",thenVsNow:"Früher vs. heute",thenNowQuestion:"Wofür stand es ursprünglich?",thenLabel:"früher",nowLabel:"heute",noLongerAcronym:"kein Akronym mehr",next:"weiter →"}
```

- [ ] **Step 4: Add the mode button**

After the `setMode('types')` button line in `renderStart`, add:

```js
            <button class="tag ${state.mode==='thenNow'?'on':''}" onclick="setMode('thenNow')">${t().thenVsNow}</button>
```

- [ ] **Step 5: Add `buildThenNowQuestions`**

Directly after the closing `}` of `buildMCQuestions`:

```js
// "Then vs. Now": the correct answer is the ORIGINAL expansion (then); today's
// expansion is planted as a trap unless the acronym is now a bare name (orphan).
// `full` carries `then` so mcAnswer/renderResults log the right expansion.
function buildThenNowQuestions(pool,n){
  return pickN(pool,n).map(a=>{
    const opts=new Set([a.then]);
    if(!a.orphan)opts.add(a.full);
    // Orphan: no `full` trap, so the answer would be the only real phrase; add another real original as decoy
    if(a.orphan){const dec=pickN(pool.filter(x=>x!==a&&!opts.has(x.then)),1)[0];if(dec)opts.add(dec.then)}
    makeDistractors(a.then,6,a.thenLang||acronymLang(a)).forEach(d=>{if(opts.size<4)opts.add(d)});
    // Short texts with little same-initial vocabulary: top up with other originals
    if(opts.size<4)pickN(pool.filter(x=>!opts.has(x.then)),4-opts.size).forEach(x=>opts.add(x.then));
    const options=shuffle([...opts]);
    return{acronym:a.acronym,full:a.then,now:a.full,orphan:!!a.orphan,cat:a.cat,subcat:a.subcat,link:a.link,options,correctIdx:options.indexOf(a.then)};
  });
}
```

- [ ] **Step 6: Branch in `startQuiz`**

After the line `if(state.mode==='glueck'){startGlueck();return}` add:

```js
  if(state.mode==='thenNow'){
    // pool is small (20), so topic/difficulty filters are ignored
    const questions=buildThenNowQuestions(ACRONYMS.filter(a=>a.then),10);
    state.quizData={questions,idx:0,score:0,streak:0,bestStreak:0,answers:[],thenNow:true};
    state.screen='mc-quiz';
    render();
    return;
  }
```

(The normal MC branch below already builds a fresh `quizData` without `thenNow`.)

- [ ] **Step 7: Question line and reveal slot in `renderMCQuiz`**

Replace

```js
          <div style="font-size:14px;color:#5c675c;margin-bottom:10px">&gt; ${t().expandAcronym}</div>
```

with

```js
          <div style="font-size:14px;color:#5c675c;margin-bottom:10px">&gt; ${q.thenNow?t().thenNowQuestion:t().expandAcronym}</div>
```

and directly after the closing `</div>` of `<div … id="options">…</div>` add:

```js
        <div id="reveal"></div>
```

- [ ] **Step 8: Reveal instead of auto-advance in `mcAnswer`**

In `mcAnswer`, replace the last line `setTimeout(()=>{q.idx++;render()},1500);` with:

```js
  if(q.thenNow){
    const now=cur.orphan?t().noLongerAcronym:cur.now;
    document.getElementById('reveal').innerHTML=`
      <div style="font-size:15px;color:#7d8a7d">${t().thenLabel}: <span style="color:#e8f5e8">${cur.full}</span> → ${t().nowLabel}: <span style="color:#e8f5e8">${now}</span></div>
      <button class="btn" style="margin-top:12px" onclick="thenNowNext()">${t().next}</button>`;
    document.getElementById('reveal').scrollIntoView({block:'nearest'});
    document.onkeydown=e=>{if(e.key==='Enter'){e.preventDefault();thenNowNext()}else if(e.key==='Escape')quitToStart()};
    return;
  }
  setTimeout(()=>{q.idx++;render()},1500);
}

// Advance past the then→now reveal. Clearing the handler first makes a fast
// double ENTER advance one question only: the second press reaches the next
// question's mcKeyHandler, which ignores ENTER while unanswered.
function thenNowNext(){
  document.onkeydown=null;
  state.quizData.idx++;
  render();
```

(The original closing `}` of `mcAnswer` now closes `thenNowNext`.)

- [ ] **Step 9: Run the mode test, verify it passes**

Run: `timeout 90 python3 $SCRATCH/thennow_mode.py "$PWD"`
Expected: `MODE OK`

Also re-run `timeout 60 python3 $SCRATCH/thennow_data.py "$PWD"` → `DATA OK`.

- [ ] **Step 10: Commit**

```bash
git add index.html
git commit -m "feat(mode): add Then vs. Now mode asking for an acronym's original meaning

<attribution lines>"
git push
```

- [ ] **Step 11: Clean up**

`rm $SCRATCH/thennow_data.py $SCRATCH/thennow_mode.py`
