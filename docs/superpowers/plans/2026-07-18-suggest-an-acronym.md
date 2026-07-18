# Suggest an Acronym Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a "Suggest an acronym" link on the start screen that opens the live Google submission form in a new tab.

**Architecture:** `game-acronym-quiz` is a single static `index.html` (React-less vanilla JS, template-string rendering, no build step, no bundler, no backend). The change is additive: one new button in `renderStart()`'s existing button row, one new i18n key in both locale objects, no new state.

**Tech Stack:** Plain JS, template strings, no framework, no package.json.

## Global Constraints

- Single-file change: `index.html` only. No new files, no dependencies, no build step — matches the rest of the codebase (verified: no `package.json`, no test framework anywhere in this repo or its sibling game repos).
- TDD adaptation: this codebase has zero test infrastructure (confirmed via `find . -iname "*test*"` — no results, and no sibling `game-*` repo has one either). Introducing a test framework for a one-button change would contradict the "follow existing patterns" rule in writing-plans. Instead, each task's "test" step is a **Playwright verification script** (ad hoc, run via the `playwright` Python package already installed in this environment — see `freaxnx01.github.io/scripts/capture_screenshots.py` for prior art) that loads the page locally, asserts the pre-change state (fails), then asserts the post-change state (passes). The script is written to `/tmp` and is NOT committed — it's a verification tool, not project test suite, consistent with the repo having no test culture.
- EN copy: `"Suggest an acronym"`. DE copy: `"Akronym vorschlagen"`. Use exactly these strings.
- Form URL (already live, do not create a new one): `https://docs.google.com/forms/d/e/1FAIpQLSenJfADHaUny0bQ6Or3YjEIDPxaq5lGRpCImNCuAAqD-q4LmQ/viewform`
- Reference GitHub issue #1 (`freaxnx01/game-acronym-quiz`) in the commit message.

---

### Task 1: Add `suggestAcronym` i18n key and the start-screen button

**Files:**
- Modify: `/home/freax/repos/github/freaxnx01/public/game-acronym-quiz/index.html:288` (en locale object)
- Modify: `/home/freax/repos/github/freaxnx01/public/game-acronym-quiz/index.html:289` (de locale object)
- Modify: `/home/freax/repos/github/freaxnx01/public/game-acronym-quiz/index.html:441-445` (start screen button row)

**Interfaces:**
- Consumes: existing `t()` i18n lookup helper (already used throughout, e.g. `t().runQuiz` at index.html:442), existing `.btn-outline` CSS class (already used at index.html:443).
- Produces: nothing consumed by later tasks — this is the only task.

- [ ] **Step 1: Write the failing verification script**

Create `/tmp/verify-suggest-acronym.py`:

```python
import sys
from playwright.sync_api import sync_playwright

PATH = "file:///home/freax/repos/github/freaxnx01/public/game-acronym-quiz/index.html"

with sync_playwright() as p:
    b = p.chromium.launch()
    page = b.new_page()
    opened_urls = []
    page.on("popup", lambda popup: opened_urls.append(popup.url))
    page.goto(PATH)
    page.wait_for_timeout(500)

    btn = page.locator("button:has-text('Suggest an acronym')")
    assert btn.count() == 1, f"expected 1 'Suggest an acronym' button, found {btn.count()}"

    with page.expect_popup() as popup_info:
        btn.click()
    popup = popup_info.value
    assert popup.url == "https://docs.google.com/forms/d/e/1FAIpQLSenJfADHaUny0bQ6Or3YjEIDPxaq5lGRpCImNCuAAqD-q4LmQ/viewform", popup.url

    page.click("button.lang-btn:has-text('DE')")
    page.wait_for_timeout(200)
    btn_de = page.locator("button:has-text('Akronym vorschlagen')")
    assert btn_de.count() == 1, f"expected 1 'Akronym vorschlagen' button in DE, found {btn_de.count()}"

    b.close()
    print("PASS")
```

- [ ] **Step 2: Run it to confirm it fails**

Run: `python3 /tmp/verify-suggest-acronym.py`
Expected: `AssertionError: expected 1 'Suggest an acronym' button, found 0`

- [ ] **Step 3: Add the i18n keys**

In `index.html:288`, insert `suggestAcronym:"Suggest an acronym",` immediately after `cancelKey:"ESC to quit"` (before the closing `}`), so the line ends:

```
...cancel:"✕ quit",cancelKey:"ESC to quit",suggestAcronym:"Suggest an acronym"},
```

In `index.html:289`, insert `suggestAcronym:"Akronym vorschlagen",` immediately after `cancelKey:"ESC zum Beenden"` (before the closing `}`), so the line ends:

```
...cancel:"✕ beenden",cancelKey:"ESC zum Beenden",suggestAcronym:"Akronym vorschlagen"}
```

- [ ] **Step 4: Add the button**

In `index.html`, the button row currently reads (lines 441-445):

```html
        <div style="display:flex;align-items:center;gap:20px;margin-top:8px;flex-wrap:wrap">
          <button class="btn" onclick="startQuiz()">▶ ${t().runQuiz}</button>
          <button class="btn-outline" onclick="state.screen='db';render()">📖 ${t().dbBrowser}</button>
          <span style="font-size:13px;color:#5c675c">10 ${t().questions} · ${t().highScore}: ${hs}/10</span>
        </div>
```

Replace it with:

```html
        <div style="display:flex;align-items:center;gap:20px;margin-top:8px;flex-wrap:wrap">
          <button class="btn" onclick="startQuiz()">▶ ${t().runQuiz}</button>
          <button class="btn-outline" onclick="state.screen='db';render()">📖 ${t().dbBrowser}</button>
          <button class="btn-outline" onclick="window.open('https://docs.google.com/forms/d/e/1FAIpQLSenJfADHaUny0bQ6Or3YjEIDPxaq5lGRpCImNCuAAqD-q4LmQ/viewform','_blank')">💡 ${t().suggestAcronym}</button>
          <span style="font-size:13px;color:#5c675c">10 ${t().questions} · ${t().highScore}: ${hs}/10</span>
        </div>
```

- [ ] **Step 5: Run the verification script to confirm it passes**

Run: `python3 /tmp/verify-suggest-acronym.py`
Expected: `PASS`

- [ ] **Step 6: Commit**

```bash
cd /home/freax/repos/github/freaxnx01/public/game-acronym-quiz
git add index.html
git commit -m "$(cat <<'EOF'
Add "Suggest an acronym" link to the start screen

Opens the live submission form in a new tab so players without a
GitHub account can propose acronyms; submissions are reviewed by
hand and added to the DB.

Refs freaxnx01/game-acronym-quiz#1
EOF
)"
```

---

## Self-Review

- **Spec coverage:** button placement (start screen, same row) ✓ Step 4. Form URL ✓ Step 4. EN/DE copy ✓ Step 3. No other UI/state changes — confirmed, only these two regions touch. Issue reference ✓ Step 6.
- **Placeholder scan:** none — every step has literal code/commands.
- **Type consistency:** n/a (no functions/types introduced, only template-string/object-literal edits).
