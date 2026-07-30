# Test what exists — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add screen 02 to `Conversational Builder v2.dc.html` — a run of the half-built journey that goes *through* the disputed step, unseen, and stops where the answer has nowhere to go.

**Architecture:** One new top-level `data-screen-label="02"` block in the source doc, three panes at 360 + 440 + 640. `build-screens.py` splits it into `screen-02.dc.html`. Three stage states driven by `?at=` off `location.search`, chosen with `sc-if` (nothing animates, so `sc-if` is legal here). The shared `Component` gains `state.at` and a set of render keys namespaced away from screen 01's.

**Tech Stack:** Static HTML with inline styles, rendered client-side by `support.js` (`<x-dc>`, `sc-if`, `style-hover` / `style-active` / `style-focus`, `DCLogic`, React). Phosphor icons from CDN, Open Sans from `fonts/`. Verification is headless Chrome screenshots plus `build-screens.py`.

**Spec:** `docs/superpowers/specs/2026-07-30-test-what-exists-design.md` — read it before Task 3.

## Global Constraints

- **Type ladder:** sizes 10 · 12 · 14 · 18 · 24 only, line-heights 12 · 16 · 18 · 22 · 24 · 28 · 38 only. **There is no 11px and no 13px.**
- **Spacing:** the FlowX scale 4 · 6 · 8 · 12 · 16 · 24 · 56. Off-scale numbers only for derived optical alignment.
- **Families:** Open Sans is plan language; `ui-monospace, SFMono-Regular, Menlo, Consolas, monospace` is everything the product says *about* the plan — counts, timestamps, identifiers, every uppercase micro-label and state chip. A micro-label is `400 10px/12px` mono, `text-transform: uppercase`, `letter-spacing: 0.14em`.
- **No elevation shadows.** `box-shadow` is only ever the 3px focus ring `0 0 0 3px rgba(43, 127, 228, 0.32)`.
- **The wash means one thing — "this is waiting on your reply."** Exactly one on screen 02: step 3's rail card. Values verbatim: `linear-gradient(180deg, rgba(242, 118, 43, 0.14), rgba(242, 118, 43, 0.06) 45%, rgba(242, 118, 43, 0.025))` behind `1px solid rgba(242, 118, 43, 0.30)`.
- **Blue is "you are here", never status:** `1px solid #4d97ea` **outline** at `outline-offset: 3px`.
- **Two question marks, one rule:** grey `ph-question-mark` at 10px marks a *mention* and takes the colour of the count it precedes (`#8b98a5` or `#5c6975`). The orange one marks the question itself — **it does not appear on screen 02 at all.**
- **`ph-question-mark`, never `ph-question`** (that is the top bar's help button).
- **Nothing offers what it cannot do:** withdraw an affordance with `pointer-events: none` in the interpolated `style` attribute **and** `tabindex="-1"`. `style-hover` / `style-active` / `style-focus` are passed to `pseudoClass()` verbatim and take **no** `{{ }}` placeholders.
- **Hover wash is `#131c24`**, including for controls standing on the orange. Never `#0b1218` — that is the ground, and it reads as a hole.
- **Palette:** ground `#0b1218`, top bar `#101820`, side pane `#0e161d`, lifted card `#131c24`, outlined card `#101820` behind `#1c2530`, hairline `#18212a`, stronger rule `#26313d`, text `#f5f9fc` / `#c3cdd7` / `#a9b6c3` / `#8b98a5` / `#5c6975` / `#4a5764`.
- **Phone palette, confined inside the bezel:** ground `#ffffff`, surface `#f4f6f8`, text `#1a2129`, secondary `#5c6975`, hairline `#dfe5ea`, action `#2b7fe4`, radius 8.
- **Transitions:** `160ms cubic-bezier(0.2, 0, 0.2, 1)` for hovers. Nothing on screen 02 animates beyond hovers and the discrete `sc-if` swap.
- **Never `sc-if` anything that animates.** Nothing on screen 02 animates, so the stage's three screens may use it.
- **Screen 01 must come out pixel-identical in both states.** It is the gate on every task.

---

### Task 1: Baseline, then the shared `Component` gains `state.at`

Screen 02 has no markup yet, so this task's deliverable is the logic plus the proof that screen 01 did not move.

**Files:**
- Create: `/private/tmp/claude-501/.../scratchpad/shot.sh` (throwaway helper, not committed)
- Modify: `Conversational Builder v2.dc.html:910-913` (the `state` initialiser) and `:926-1126` (`renderVals()`)

**Interfaces:**
- Consumes: nothing.
- Produces: render keys used by Tasks 3–5 — `at1` / `at2` / `at3` (booleans for `sc-if`), `goTo1` / `goTo2` / `goTo3` (setters), `selRow1` / `selRow2` / `selRow3` (blue outline colour strings), `rowPE4` (always `'none'`), `rowTab4` (always `'-1'`), `runUpPE` (always `'auto'`), `runExitTab` (always `'0'`), `stageCap` (the mono caption string).

- [ ] **Step 1: Start the server and capture the screen 01 baseline**

```bash
cd "/Users/bogdandraghici/Desktop/vibes/AI Assisted Builder"
python3 -m http.server 8765 --bind 127.0.0.1 &
sleep 1
S="/private/tmp/claude-501/-Users-bogdandraghici-Desktop-vibes-AI-Assisted-Builder/e432d9e7-5cf0-462a-bae0-158e5751fd0d/scratchpad"
CHROME="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
"$CHROME" --headless --disable-gpu --hide-scrollbars --force-device-scale-factor=1 \
  --window-size=1440,900 --virtual-time-budget=6000 \
  --screenshot="$S/base-01-plan.png" "http://127.0.0.1:8765/screen-01.dc.html"
"$CHROME" --headless --disable-gpu --hide-scrollbars --force-device-scale-factor=1 \
  --window-size=1440,900 --virtual-time-budget=6000 \
  --screenshot="$S/base-01-step.png" "http://127.0.0.1:8765/screen-01.dc.html?step=3"
```

Expected: two 1440×900 PNGs. Sanity-check them by eye — if either shows a `.sc-placeholder` or an empty pane, the virtual-time budget was too short; raise it and retake. **A baseline captured off a hydrating page is worse than no baseline** (see `docs/motion.md`, "Traps in headless Chrome").

- [ ] **Step 2: Add `at` to the state initialiser**

Replace lines 910–913:

```js
  state = {
    step: new URLSearchParams(location.search).get('step') === '3',
    tech: false
  };
```

with:

```js
  // Screen 01 is two states of one screen; screen 02 is three states of
  // another, and they share this class because every screen in the source doc
  // does. The keys stay disjoint: `step`/`tech` are screen 01's, `at` is
  // screen 02's. `?at=` is read exactly the way `?step=` is, so index.html can
  // show a second state of screen 02 without a second copy of its markup.
  state = {
    step: new URLSearchParams(location.search).get('step') === '3',
    tech: false,
    at: Math.min(3, Math.max(1, parseInt(new URLSearchParams(location.search).get('at'), 10) || 3))
  };
```

- [ ] **Step 3: Add screen 02's render values**

Insert immediately before the closing `};` of `renderVals()` (after the `planCrumbColor` line at 1125), keeping the leading comma on the previous line:

```js
      // ── Screen 02: the run, and where it stops ───────────────────────────
      // Three stage states, chosen with sc-if. That is legal here and only
      // here: nothing on screen 02 animates, and the three phone screens are
      // genuinely different content rather than one thing changing shape, so
      // the "one element per thing" rule does not apply. An instant swap is
      // also the honest reading — it is a different screen, not a transform.
      at1: this.state.at === 1,
      at2: this.state.at === 2,
      at3: this.state.at === 3,
      goTo1: () => this.setState({ at: 1 }),
      goTo2: () => this.setState({ at: 2 }),
      goTo3: () => this.setState({ at: 3 }),

      // Blue is "you are here": the rail row whose screen the stage is
      // showing. On row 3 at rest it stacks with the wash, which is the
      // orthogonality the rule already allows — selection and "your move" are
      // different questions about the same card.
      selRow1: this.state.at === 1 ? '#4d97ea' : 'transparent',
      selRow2: this.state.at === 2 ? '#4d97ea' : 'transparent',
      selRow3: this.state.at === 3 ? '#4d97ea' : 'transparent',

      // Rows 4 and 5 are not steps you can look at — nothing of them exists to
      // show. One flag, whole affordance: the pointer takes the hover and the
      // cursor with it, and the tabindex keeps the keyboard from reaching what
      // the mouse cannot.
      rowPE4: 'none',
      rowTab4: '-1',

      // Up from a run IS the plan, and unlike the journey list it exists — so
      // unlike screen 01's plan state, these are live. They are constants
      // rather than expressions because screen 02 has one answer in all three
      // of its states; they exist as keys so the markup reads the same as
      // screen 01's and the reason lives here.
      runUpPE: 'auto',
      runExitTab: '0',

      // What the phone cannot say about itself. Mono, because it is the
      // product talking about the run rather than the plan talking.
      stageCap: this.state.at === 1
        ? 'step 1 · they picked one of twelve'
        : this.state.at === 2
          ? 'step 2 · twenty-two words, then Continue'
          : 'step 3 · the check ran behind this screen · unseen · 3s'
```

- [ ] **Step 4: Regenerate and prove screen 01 has not moved**

```bash
cd "/Users/bogdandraghici/Desktop/vibes/AI Assisted Builder"
python3 build-screens.py
```

Expected: `wrote screen-01.dc.html  (Screen 01 — the plan, and the step you open beside it)` and nothing else. Then re-shoot both screen 01 states to `after-01-plan.png` / `after-01-step.png` with the same command as Step 1 and compare:

```bash
python3 - <<'PY'
import pathlib, sys
S = pathlib.Path("/private/tmp/claude-501/-Users-bogdandraghici-Desktop-vibes-AI-Assisted-Builder/e432d9e7-5cf0-462a-bae0-158e5751fd0d/scratchpad")
for name in ("01-plan", "01-step"):
    a = (S / f"base-{name}.png").read_bytes()
    b = (S / f"after-{name}.png").read_bytes()
    print(name, "IDENTICAL" if a == b else f"DIFFERS ({len(a)} vs {len(b)} bytes)")
PY
```

Expected: `01-plan IDENTICAL` and `01-step IDENTICAL`. A byte-identical PNG is a stronger gate than a pixel diff and needs no extra tooling. If either differs, the new keys collided with an existing one or the initialiser edit changed screen 01's starting state — fix before continuing.

- [ ] **Step 5: Commit**

```bash
git add "Conversational Builder v2.dc.html" screen-01.dc.html
git commit -m "run: the shared logic learns a third state, and screen 01 does not move"
```

---

### Task 2: The screen 02 block — caption, frame, top bar, three empty panes

**Files:**
- Modify: `Conversational Builder v2.dc.html` — insert a caption block and a screen block between screen 01's closing `  </div>` (line 891) and the canvas's closing `</div>` (line 893)

**Interfaces:**
- Consumes: `runUpPE`, `runExitTab` from Task 1.
- Produces: three empty pane divs Tasks 3–5 fill — the chat (`width: 360px`), the rail (`width: 440px`), the stage (`flex: 1`).

- [ ] **Step 1: Add the caption block**

`build-screens.py` finds captions by an exact string match on `CAPTION_OPEN` and pulls the title with `>(Screen\s+[^<]+)<`, so the opening tag and the title's prefix are load-bearing. Insert after line 891:

```html
  <div style="width: 1440px; display: flex; flex-direction: column; gap: 14px;">
    <div style="display: flex; align-items: baseline; gap: 16px;">
      <div style="font: 400 10px/12px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; text-transform: uppercase; letter-spacing: 0.14em; color: #8b98a5; flex: none;">Screen 02 — the run, and the edge of what exists</div>
      <div style="flex: 1; height: 1px; background: #18212a;"></div>
      <div style="font: 400 12px/16px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; color: #8b98a5; flex: none;">1440 × 900 · live</div>
    </div>
    <div style="font: 400 14px/24px 'Open Sans', sans-serif; color: #a9b6c3; max-width: 104ch; text-wrap: pretty;">Two of five steps are built, so this run cannot finish — which makes the edge, not the preview, the subject. The check you asked to drop was built at 09:37, two minutes before you objected, so the run goes straight through it: unseen, three seconds, never in the customer's way. Then it stops, because the merchant's answer has nowhere to go until the evidence step exists.</div>
    <div style="font: 400 14px/24px 'Open Sans', sans-serif; color: #a9b6c3; max-width: 104ch; text-wrap: pretty;">So the run is the argument's evidence, and the act comes with it: step 3 keeps the wash and carries <strong style="font-weight: 600; color: #f5f9fc;">Agree</strong> here, where you have just watched the disputed step run. <strong style="font-weight: 600; color: #f5f9fc;">Click the first two rows</strong> to scrub the run back — the rest cannot be looked at, because nothing of them exists to show.</div>
  </div>
```

- [ ] **Step 2: Add the screen block's opening tag and closing tag**

The opening tag must contain `width: 1440px; height: 900px;`, `border: 1px solid #26313d; border-radius: 12px; ` and `flex: none; ` **verbatim** — `build-screens.py` hard-fails otherwise. Copy screen 01's opening tag at line 42 exactly, changing only the label:

```html
  <div data-screen-label="02" style="width: 1440px; height: 900px; background: #0b1218; border: 1px solid #26313d; border-radius: 12px; overflow: hidden; display: flex; flex-direction: column; flex: none; color: #f5f9fc;">
```

and close it with `  </div>` at column 2 (the generator's `BLOCK_CLOSE` is the exact string `  </div>`, so the indentation matters).

- [ ] **Step 3: Add the top bar**

Copy screen 01's top bar (lines 44–89) and change exactly five things. Everything else — the mark, the undo pair, the kebab, the divider, the bell, the help `ph-question`, the `LC` avatar — comes across untouched.

1. The back arrow: replace every `{{ up* }}` and `{{ exitTab }}` with the constants, since screen 02 has one answer in all its states:

```html
        <div onClick="{{ goTo3 }}" tabindex="{{ runExitTab }}" title="Back to the plan" style="width: 26px; height: 26px; border-radius: 6px; border: 1px solid #26313d; background: #161f28; display: inline-flex; align-items: center; justify-content: center; color: #c3cdd7; cursor: pointer; flex: none; outline: none; pointer-events: {{ runUpPE }}; transition: background 160ms cubic-bezier(0.2, 0, 0.2, 1);" style-hover="background: #1c2632; color: #f5f9fc;" style-focus="box-shadow: 0 0 0 3px rgba(43, 127, 228, 0.32);">
          <i class="ph ph-arrow-left" style="font-size: 14px;"></i>
        </div>
```

`onClick="{{ goTo3 }}"` rather than a navigation: leaving the doc is Task 6's job via `index.html`, and a control that resets the run to its edge is at least honest about what it does here.

2. The breadcrumb: four crumbs, no animated `crumbCols` grid — screen 02 has no collapse. `The plan` is `#c3cdd7` and live; `Test run` is the white semibold "where you are".

```html
        <div style="display: flex; align-items: center; gap: 8px; font: 400 12px/16px 'Open Sans', sans-serif; min-width: 0;">
          <span style="color: #8b98a5;">Journeys</span>
          <span style="color: #4a5764;">/</span>
          <span style="color: #8b98a5;">Card payment dispute</span>
          <span style="color: #4a5764;">/</span>
          <span onClick="{{ goTo3 }}" tabindex="{{ runExitTab }}" title="Back to the plan" style="color: #c3cdd7; font-weight: 600; cursor: pointer; border-radius: 4px; outline: none; pointer-events: {{ runUpPE }}; transition: color 160ms cubic-bezier(0.2, 0, 0.2, 1);" style-hover="color: #ffffff;" style-focus="box-shadow: 0 0 0 3px rgba(43, 127, 228, 0.32);">The plan</span>
          <span style="color: #4a5764;">/</span>
          <span style="color: #ffffff; font-weight: 600;">Test run</span>
        </div>
```

3. The strapline becomes the run's identity in the same slot:

```html
          <div style="font: 400 12px/16px 'Open Sans', sans-serif; color: #c3cdd7;">ran build 2 · 09:46 · reached 3 of 5</div>
```

4. The green button's label becomes `Run again` (same `#1d9a5c`, same `ph-play`, same hover `#2ea86a`, same focus ring `rgba(46, 168, 106, 0.32)`).

5. Nothing else. In particular the help icon stays `ph-question` — it is the help button, and the mention glyph is `ph-question-mark`.

- [ ] **Step 4: Add the three empty panes**

```html
    <div style="flex: 1; display: flex; min-height: 0;">
      <div style="width: 360px; flex: none; background: #0b1218; border-right: 1px solid #18212a; display: flex; flex-direction: column; min-height: 0;"></div>
      <div style="width: 440px; flex: none; background: #0b1218; border-right: 1px solid #18212a; display: flex; flex-direction: column; min-height: 0; padding: 20px 24px 16px;"></div>
      <div style="flex: 1; min-width: 0; background: #0e161d; display: flex; align-items: center; justify-content: center; padding: 24px 0;"></div>
    </div>
```

The stage takes the side-pane `#0e161d` rather than the ground: it is the one pane holding an object rather than a list, and the shade separates the phone from the chrome without a border. `flex: 1` on the stage rather than a fixed 640 so a viewport wider than 1440 gives the extra room to the stage, which is what screen 01's step pane does.

- [ ] **Step 5: Regenerate, verify, commit**

```bash
cd "/Users/bogdandraghici/Desktop/vibes/AI Assisted Builder"
python3 build-screens.py
```

Expected: **two** `wrote` lines — `screen-01.dc.html` and `screen-02.dc.html  (Screen 02 — the run, and the edge of what exists)`. If the title falls back to `Screen 02`, the caption's `CAPTION_OPEN` string does not match exactly.

Re-run the Task 1 Step 4 byte-comparison for screen 01 — still `IDENTICAL` both states. Then shoot screen 02 and check by eye: top bar with four crumbs and `Run again`, three panes at 360 / 440 / rest, no scrollbar.

```bash
git add "Conversational Builder v2.dc.html" screen-01.dc.html screen-02.dc.html
git commit -m "run: a second screen, three panes and a bar that knows where it is"
```

---

### Task 3: The chat — one more turn, and the composer's promise sharpened

**Files:**
- Modify: `Conversational Builder v2.dc.html` — fill screen 02's 360px pane

**Interfaces:**
- Consumes: nothing from Task 1.
- Produces: nothing later tasks read.

- [ ] **Step 1: Copy screen 01's chat pane body wholesale**

Copy lines 100–160 of screen 01 — the scrolling body with its six blocks (09:29 user, 09:31 agent, the "4 turns about wording" divider, 09:39 user, 09:41 agent with its three sub-blocks and the `Scheme rules §4.2` citation, 09:44 user) and the composer block. **Verbatim**: the two chat panes drifting apart has already been a bug here twice (`817453c`, `a9cbc40`), and the only defence is that they are the same text.

- [ ] **Step 2: Add the agent's run report as a seventh block**

After the 09:44 user block, inside the scrolling body:

```html
          <div style="display: flex; gap: 12px;">
            <div style="width: 22px; height: 22px; flex: none; border-radius: 99px; background: rgba(139, 152, 165, 0.16); display: grid; place-items: center;"><img src="assets/flowx-mark-white.svg" alt="Agent" style="height: 11px; width: auto; flex: none;" /></div>
            <div style="flex: 1; min-width: 0; display: flex; flex-direction: column; gap: 4px;">
              <div style="font: 600 12px/16px 'Open Sans', sans-serif; color: #8b98a5;">Agent <span style="font-weight: 400;">· 09:46</span></div>
              <div style="font: 400 14px/24px 'Open Sans', sans-serif; color: #f5f9fc; text-wrap: pretty;">I ran what exists. Both built steps, then the check — which went out while the customer was still typing and came back in three seconds. It stops there.</div>
            </div>
          </div>
```

Four lines at the pane's 278px measure. The pane is `overflow: hidden` with no scroller, and the measured budget is 723px against 676px of content — so **this copy may not grow past five lines.** If it needs to say more, shorten it; do not add a scroller.

- [ ] **Step 3: Change the composer's caption**

In the copied composer block, replace

```html
          <div style="font: 400 12px/16px 'Open Sans', sans-serif; color: #8b98a5;">Anything you type lands on the step it is about.</div>
```

with

```html
          <!-- On the plan the agent has to infer which step you mean; here you
               are looking at one, so it lands there. Same promise, one degree
               more specific — which is the whole yield of a run: what you
               noticed becomes plan language on the right step. -->
          <div style="font: 400 12px/16px 'Open Sans', sans-serif; color: #8b98a5;">Anything you type lands on the step you are looking at.</div>
```

- [ ] **Step 4: Regenerate, verify, commit**

Run `python3 build-screens.py`, re-run the screen 01 byte-comparison (both `IDENTICAL`), then shoot screen 02 and confirm: seven turns visible top to bottom, nothing clipped at the top, the composer and its new caption at the bottom.

```bash
git add "Conversational Builder v2.dc.html" screen-01.dc.html screen-02.dc.html
git commit -m "run: the agent says what it ran, and the composer knows where you are looking"
```

---

### Task 4: The rail — the plan, told in run terms, with the act at the edge

The largest task, and the one carrying every colour rule. Read the spec's "The edge, and the act" and "Colour discipline" sections before starting.

**Files:**
- Modify: `Conversational Builder v2.dc.html` — fill screen 02's 440px pane

**Interfaces:**
- Consumes: `selRow1` / `selRow2` / `selRow3`, `goTo1` / `goTo2` / `goTo3`, `rowPE4`, `rowTab4` from Task 1.
- Produces: nothing later tasks read.

**Shared row shape.** Every row is `display: flex; gap: 16px; align-items: stretch;` with a 20px dot gutter (dot 20px, then a 2px connector filling the rest) and a `flex: 1; min-width: 0` content column at 356px. Titles are `600 14px/22px 'Open Sans'`, `text-wrap: pretty`. Under each title, stacked and never beside it, comes the mono run line — the counts-column rule from `docs/motion.md`, applied here because a column beside the title narrows it.

- [ ] **Step 1: The header**

```html
      <div style="flex: none; display: flex; flex-direction: column; gap: 6px; padding-bottom: 16px;">
        <div style="font: 700 18px/28px 'Open Sans', sans-serif; color: #f5f9fc;">The plan</div>
        <div style="display: flex; align-items: center; gap: 12px;">
          <div style="display: flex; align-items: center; gap: 3px; flex: none;">
            <div style="width: 18px; height: 3px; border-radius: 2px; background: #2ea86a;"></div>
            <div style="width: 18px; height: 3px; border-radius: 2px; background: #2ea86a;"></div>
            <div style="width: 18px; height: 3px; border-radius: 2px; background: #f2762b;"></div>
            <div style="width: 18px; height: 3px; border-radius: 2px; background: #33404d;"></div>
            <div style="width: 18px; height: 3px; border-radius: 2px; background: #33404d;"></div>
          </div>
        </div>
        <div style="font: 400 12px/16px 'Open Sans', sans-serif; color: #8b98a5;">Two of five built · one not agreed · <i class="ph ph-question-mark" style="font-size: 10px;"></i> four to settle</div>
      </div>
```

Same words as the wide plan's subtitle and the same dash bar, because it is the same object — a third name for it would read as a change of subject. The subtitle drops to its own line because at 440 it does not fit beside the bar.

- [ ] **Step 2: The three group headings**

One micro-label each, in the colours screen 01's spine uses. Insert each above its rows:

```html
      <div style="flex: none; padding-bottom: 8px; font: 400 10px/12px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; text-transform: uppercase; letter-spacing: 0.14em; color: #2ea86a;">Agreed</div>
```

then `#f2762b` / `Not agreed` before row 3, and `#5c6975` / `Next` before row 4.

- [ ] **Step 3: Rows 1 and 2 — ran, and selectable**

Row 1. The whole content column is the click target and it carries the blue outline, so selection and the affordance are the same object:

```html
      <div style="flex: none; display: flex; gap: 16px; align-items: stretch;">
        <div style="width: 20px; flex: none; display: flex; flex-direction: column; align-items: center;">
          <div style="width: 20px; height: 20px; flex: none; border-radius: 99px; background: #2ea86a; display: grid; place-items: center;"><i class="ph ph-check" style="font-size: 11px; color: #ffffff;"></i></div>
          <div style="flex: 1; width: 2px; background: #26313d; margin-top: 4px;"></div>
        </div>
        <div onClick="{{ goTo1 }}" tabindex="0" style="flex: 1; min-width: 0; margin: -6px -8px 0; padding: 6px 8px; border-radius: 8px; cursor: pointer; outline: 1px solid {{ selRow1 }}; outline-offset: 3px; display: flex; flex-direction: column; gap: 2px; transition: background 160ms cubic-bezier(0.2, 0, 0.2, 1), outline-color 160ms cubic-bezier(0.2, 0, 0.2, 1);" style-hover="background: #131c24;" style-focus="box-shadow: 0 0 0 3px rgba(43, 127, 228, 0.32);">
          <div style="font: 600 14px/22px 'Open Sans', sans-serif; color: #f5f9fc; text-wrap: pretty;">The customer picks the transaction they do not recognise</div>
          <div style="font: 400 12px/16px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; color: #8b98a5;">ran · picked 1 of 12</div>
        </div>
      </div>
      <div style="flex: none; height: 20px;"></div>
```

The 20px spacer stands in for screen 01's `rowPb`, which cannot live on the row here because the outline would enclose it.

Row 2 is the same shape with `{{ goTo2 }}`, `{{ selRow2 }}`, the title `We ask what happened, in their words` and the mono line `ran · 22 words`.

- [ ] **Step 4: Row 3 — the wash, the run's finding, and the act**

The one wash on this screen. It stacks the blue outline with the wash's own orange border, which is the orthogonality the palette doc already permits.

```html
      <div style="flex: none; display: flex; gap: 16px; align-items: stretch;">
        <div style="width: 20px; flex: none; display: flex; flex-direction: column; align-items: center;">
          <div style="width: 20px; height: 20px; flex: none; border-radius: 99px; background: #f2762b; display: grid; place-items: center;"><i class="ph ph-warning" style="font-size: 11px; color: #ffffff;"></i></div>
          <div style="flex: 1; width: 0; border-left: 2px dashed #26313d; margin-top: 4px;"></div>
        </div>
        <div onClick="{{ goTo3 }}" tabindex="0" style="flex: 1; min-width: 0; background: linear-gradient(180deg, rgba(242, 118, 43, 0.14), rgba(242, 118, 43, 0.06) 45%, rgba(242, 118, 43, 0.025)); border: 1px solid rgba(242, 118, 43, 0.30); border-radius: 12px; padding: 16px; cursor: pointer; outline: 1px solid {{ selRow3 }}; outline-offset: 3px; display: flex; flex-direction: column; gap: 6px; transition: outline-color 160ms cubic-bezier(0.2, 0, 0.2, 1);" style-focus="box-shadow: 0 0 0 3px rgba(43, 127, 228, 0.32);">
          <div style="font: 600 14px/22px 'Open Sans', sans-serif; color: #f5f9fc; text-wrap: pretty;">We check with the merchant before asking for evidence</div>
          <!-- The claim under dispute was "it slows the customer down". This
               line is that claim, observed instead of argued: the drawing's
               `unseen · 3s` is now what happened. -->
          <div style="font: 400 12px/16px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; color: #8b98a5;">ran · unseen · 3s</div>
          <div style="height: 1px; background: rgba(242, 118, 43, 0.16); margin: 6px 0;"></div>
          <!-- The same pair, the same size, the same glyphs and the same
               caption as the plan card and the step pane's pinned footer. A
               third view of one object says the same thing in the same words,
               or it reads as a change of subject. It is washed here because
               this is where you now have the best reason to answer: you have
               just watched the disputed step run without touching the
               customer. Wash it where the reply is given. -->
          <div style="display: flex; align-items: center; gap: 12px;">
            <div tabindex="0" style="display: inline-flex; flex: none; white-space: nowrap; align-items: center; gap: 6px; height: 36px; padding: 0 12px; font: 600 14px/24px 'Open Sans', sans-serif; color: #ffffff; background: #2b7fe4; border-radius: 8px; cursor: pointer; outline: none; transition: background 160ms cubic-bezier(0.2, 0, 0.2, 1);" style-hover="background: #4d97ea;" style-active="background: #2b5f9e;" style-focus="box-shadow: 0 0 0 3px rgba(43, 127, 228, 0.32);"><i class="ph ph-check" style="font-size: 14px;"></i>Agree</div>
            <div tabindex="0" style="display: inline-flex; flex: none; white-space: nowrap; align-items: center; gap: 6px; height: 36px; padding: 0 12px; font: 600 14px/24px 'Open Sans', sans-serif; color: #c3cdd7; background: transparent; border: 1px solid #26313d; border-radius: 8px; cursor: pointer; outline: none; transition: all 160ms cubic-bezier(0.2, 0, 0.2, 1);" style-hover="border-color: #4d97ea; background: #131c24; color: #f5f9fc;" style-focus="box-shadow: 0 0 0 3px rgba(43, 127, 228, 0.32);"><i class="ph ph-chat-teardrop-text" style="font-size: 14px;"></i>Comment further</div>
          </div>
          <div style="display: flex; align-items: center; gap: 4px; font: 400 12px/16px 'Open Sans', sans-serif; color: #8b98a5;"><i class="ph ph-question-mark" style="font-size: 10px;"></i>One question still to settle before I build it.</div>
        </div>
      </div>
      <div style="flex: none; height: 20px;"></div>
```

The card has no `style-hover`: it already wears a wash and two live buttons, and a hover wash over a card whose interior is the affordance says the whole card does something different from what its buttons do.

- [ ] **Step 5: The boundary**

Between row 3 and the `Next` heading. Grey, never orange — it is a fact about the build, not your move. And it names **no resource**: `enquiryToContext.dataMapper` is behind a disclosure on screen 01 and stays there.

```html
      <div style="flex: none; display: flex; flex-direction: column; gap: 8px; padding-bottom: 24px;">
        <div style="display: flex; align-items: center; gap: 8px;">
          <div style="font: 400 10px/12px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; text-transform: uppercase; letter-spacing: 0.14em; color: #5c6975; flex: none;">What exists ends here</div>
          <div style="flex: 1; height: 1px; background: #26313d;"></div>
        </div>
        <div style="font: 400 12px/18px 'Open Sans', sans-serif; color: #8b98a5; text-wrap: pretty;">The merchant's answer has nowhere to go until the evidence step exists.</div>
      </div>
```

- [ ] **Step 6: Rows 4 and 5 — not built, and inert**

Same row shape, but the content column takes `pointer-events: {{ rowPE4 }}` and `tabindex="{{ rowTab4 }}"`, no `onClick`, no `cursor`, no `style-hover` and no outline. Nothing of these steps exists to show, so nothing here may offer to show it — and the pointer takes the hover and the cursor away together while the tabindex keeps the keyboard from reaching what the mouse cannot.

Row 4: dot is `width: 20px; height: 20px; border-radius: 99px; background: #0b1218; border: 1px dashed #8b98a5;` with a dashed connector. Title `We ask for evidence, only what the reason needs` in `#c3cdd7`. Then two stacked mono lines — `not built` in `#5c6975`, and the question mention:

```html
          <div style="font: 400 12px/16px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; color: #5c6975;">not built</div>
          <div style="display: flex; align-items: center; gap: 4px; font: 400 10px/12px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; color: #5c6975;"><i class="ph ph-question-mark" style="font-size: 10px;"></i>2 to settle</div>
```

Row 5: dot is `background: #0b1218; border: 1px solid #8b98a5;` with `<i class="ph ph-user" style="font-size: 11px; color: #c3cdd7;"></i>` inside. Title `Disputes over {{ threshold }} go to a person` in `#f5f9fc` — the `{{ threshold }}` prop already exists and defaults to `£500`. Mono `not built`, then `1 to settle`.

- [ ] **Step 7: The trailing note, then let the rail breathe**

```html
      <div style="flex: none; display: flex; gap: 16px; align-items: flex-start; padding-top: 4px;">
        <div style="width: 20px; flex: none; display: flex; justify-content: center; padding-top: 5px;">
          <div style="width: 8px; height: 8px; border-radius: 99px; border: 1px dashed #8b98a5;"></div>
        </div>
        <div style="flex: 1; min-width: 0; font: 400 12px/18px 'Open Sans', sans-serif; color: #8b98a5; text-wrap: pretty;">Two more steps will be needed — how the reviewer decides, and what the customer is told. I have not written them yet.</div>
      </div>
      <div style="flex: 1;"></div>
```

- [ ] **Step 8: Verify the rail fits, then commit**

Regenerate, re-run the screen 01 byte-comparison, then shoot screen 02 and check **specifically**:

1. **No scrollbar and no clipping in the rail.** The budget is 811 of 852. If the last row or the trailing note is cut off, reduce the 20px inter-row spacers to 16 before touching type sizes.
2. **Exactly one wash** — row 3 and nothing else.
3. **Blue outline on row 3 only** at the default `at=3`; then shoot `?at=1` and confirm it has moved to row 1 and row 3 keeps its orange border alone.
4. **Rows 4 and 5 do not respond to hover** and have no pointer cursor.

```bash
git add "Conversational Builder v2.dc.html" screen-01.dc.html screen-02.dc.html
git commit -m "run: the plan in run terms, and the act where the evidence is"
```

---

### Task 5: The stage — a phone, three screens, and the tool drawing over the app

**Files:**
- Modify: `Conversational Builder v2.dc.html` — fill screen 02's stage pane

**Interfaces:**
- Consumes: `at1` / `at2` / `at3`, `goTo2` / `goTo3`, `stageCap` from Task 1.
- Produces: nothing.

- [ ] **Step 1: The bezel and the caption slot**

```html
      <div style="flex: 1; min-width: 0; background: #0e161d; display: flex; align-items: center; justify-content: center; padding: 24px 0;">
        <div style="flex: none; display: flex; flex-direction: column; gap: 12px;">
          <!-- The bezel's 32px radius is hardware geometry, not the UI radius
               scale — the same exemption the icon sizes get from the type
               ladder. It is the only off-scale radius on this screen, and it
               exists because a 360px slab at 12px does not read as a phone.
               No shadow: there are none in this system, and the bezel's own
               hairline is the whole separation it needs. -->
          <div style="width: 360px; height: 760px; flex: none; background: #ffffff; border: 1px solid #26313d; border-radius: 32px; overflow: hidden; position: relative; display: flex; flex-direction: column;">
            <!-- the app's status bar, then the three screens -->
          </div>
          <div style="font: 400 12px/16px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; color: #8b98a5;">{{ stageCap }}</div>
        </div>
      </div>
```

- [ ] **Step 2: The status bar, in the app's own palette**

First child of the bezel, above the three screens — the same on all three, so it sits outside every `sc-if`:

```html
            <div style="flex: none; height: 44px; padding: 0 24px; display: flex; align-items: center; justify-content: space-between; font: 600 12px/16px 'Open Sans', sans-serif; color: #1a2129;">
              <div>09:46</div>
              <div style="display: flex; align-items: center; gap: 6px;"><i class="ph ph-cell-signal-full" style="font-size: 12px;"></i><i class="ph ph-wifi-high" style="font-size: 12px;"></i><i class="ph ph-battery-full" style="font-size: 12px;"></i></div>
            </div>
```

- [ ] **Step 3: Screen 1 of the app — "Which payment?"**

Wrapped in `<sc-if value="{{ at1 }}" hint-placeholder-val="{{ false }}">`. One transaction row is live and advances to 2; the other three are stills with **no hover, no cursor and no tabindex**, because the run only ever went one way.

```html
<sc-if value="{{ at1 }}" hint-placeholder-val="{{ false }}">
            <div style="flex: 1; min-height: 0; padding: 8px 20px 20px; display: flex; flex-direction: column; gap: 16px;">
              <div style="font: 700 18px/28px 'Open Sans', sans-serif; color: #1a2129;">Which payment?</div>
              <div style="font: 400 14px/22px 'Open Sans', sans-serif; color: #5c6975;">Pick the one you do not recognise.</div>
              <div style="display: flex; flex-direction: column; gap: 8px;">
                <div onClick="{{ goTo2 }}" style="display: flex; align-items: center; justify-content: space-between; gap: 12px; padding: 12px; background: #f4f6f8; border: 1px solid #2b7fe4; border-radius: 8px; cursor: pointer;">
                  <div style="min-width: 0;">
                    <div style="font: 600 14px/22px 'Open Sans', sans-serif; color: #1a2129;">NORTHGATE SUPPLY CO</div>
                    <div style="font: 400 12px/18px 'Open Sans', sans-serif; color: #5c6975;">24 July · card ending 4417</div>
                  </div>
                  <div style="font: 600 14px/22px 'Open Sans', sans-serif; color: #1a2129; flex: none;">£248.00</div>
                </div>
                <div style="display: flex; align-items: center; justify-content: space-between; gap: 12px; padding: 12px; background: #ffffff; border: 1px solid #dfe5ea; border-radius: 8px;">
                  <div style="min-width: 0;">
                    <div style="font: 600 14px/22px 'Open Sans', sans-serif; color: #1a2129;">CAFE MARGUERITE</div>
                    <div style="font: 400 12px/18px 'Open Sans', sans-serif; color: #5c6975;">24 July · card ending 4417</div>
                  </div>
                  <div style="font: 600 14px/22px 'Open Sans', sans-serif; color: #1a2129; flex: none;">£6.40</div>
                </div>
                <div style="display: flex; align-items: center; justify-content: space-between; gap: 12px; padding: 12px; background: #ffffff; border: 1px solid #dfe5ea; border-radius: 8px;">
                  <div style="min-width: 0;">
                    <div style="font: 600 14px/22px 'Open Sans', sans-serif; color: #1a2129;">TFL TRAVEL</div>
                    <div style="font: 400 12px/18px 'Open Sans', sans-serif; color: #5c6975;">23 July · card ending 4417</div>
                  </div>
                  <div style="font: 600 14px/22px 'Open Sans', sans-serif; color: #1a2129; flex: none;">£4.80</div>
                </div>
              </div>
              <div style="font: 400 12px/18px 'Open Sans', sans-serif; color: #5c6975;">Showing the last ninety days.</div>
            </div>
</sc-if>
```

`Showing the last ninety days.` is the step's own description from the plan (*"Last ninety days. Card and account already known, so nothing is typed twice."*) said the way the app would say it — the run showing the plan kept its word.

- [ ] **Step 4: Screen 2 of the app — "What happened?"**

Wrapped in `<sc-if value="{{ at2 }}" hint-placeholder-val="{{ false }}">`. `Continue` is live and advances to 3.

```html
<sc-if value="{{ at2 }}" hint-placeholder-val="{{ false }}">
            <div style="flex: 1; min-height: 0; padding: 8px 20px 20px; display: flex; flex-direction: column; gap: 16px;">
              <div style="font: 700 18px/28px 'Open Sans', sans-serif; color: #1a2129;">What happened?</div>
              <div style="font: 400 14px/22px 'Open Sans', sans-serif; color: #5c6975;">In your own words. We will work out the rest.</div>
              <div style="flex: none; min-height: 120px; padding: 12px; background: #ffffff; border: 1px solid #2b7fe4; border-radius: 8px; font: 400 14px/22px 'Open Sans', sans-serif; color: #1a2129;">I ordered a parcel from them on the 24th and it never turned up. They stopped replying to my emails after the second one.</div>
              <div style="font: 400 12px/18px 'Open Sans', sans-serif; color: #5c6975;">NORTHGATE SUPPLY CO · £248.00 · 24 July</div>
              <div style="flex: 1;"></div>
              <div onClick="{{ goTo3 }}" style="flex: none; height: 44px; border-radius: 8px; background: #2b7fe4; color: #ffffff; display: grid; place-items: center; font: 600 14px/22px 'Open Sans', sans-serif; cursor: pointer;">Continue</div>
            </div>
</sc-if>
```

- [ ] **Step 5: Screen 3 — the edge, with the tool drawing over the app**

Wrapped in `<sc-if value="{{ at3 }}" hint-placeholder-val="{{ true }}">` — the default. The same markup as screen 2 but with `Continue` **inert** (no `onClick`, no cursor), plus the veil and the panel. Copy screen 2's block, remove the `onClick` and `cursor: pointer` from `Continue`, and add as the bezel's last child:

```html
            <div style="position: absolute; inset: 0; background: rgba(11, 18, 24, 0.72); display: flex; align-items: center; justify-content: center; padding: 32px;">
              <!-- Dark, inside a white app, on purpose: white is the thing you
                   built and dark is the tool speaking. It carries no wash — it
                   is a fact about the build, not your move — and no way
                   forward, because there is not one. -->
              <div style="width: 100%; background: #101820; border: 1px solid #26313d; border-radius: 12px; padding: 16px; display: flex; flex-direction: column; gap: 8px;">
                <div style="font: 400 10px/12px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; text-transform: uppercase; letter-spacing: 0.14em; color: #8b98a5;">What exists ends here</div>
                <div style="font: 400 12px/18px 'Open Sans', sans-serif; color: #c3cdd7; text-wrap: pretty;">The customer would go on to send a receipt. That step is not built.</div>
              </div>
            </div>
```

- [ ] **Step 6: Verify all three states, then commit**

Regenerate, re-run the screen 01 byte-comparison, then shoot all three:

```bash
S="/private/tmp/claude-501/-Users-bogdandraghici-Desktop-vibes-AI-Assisted-Builder/e432d9e7-5cf0-462a-bae0-158e5751fd0d/scratchpad"
CHROME="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
for a in 1 2 3; do
  "$CHROME" --headless --disable-gpu --hide-scrollbars --force-device-scale-factor=1 \
    --window-size=1440,900 --virtual-time-budget=6000 \
    --screenshot="$S/s02-at$a.png" "http://127.0.0.1:8765/screen-02.dc.html?at=$a"
done
```

Check: the bezel is not clipped top or bottom; the caption under it matches the state; the veil and panel appear only at `at=3`; `Continue` is blue in all three but only hoverable at `at=2`; the light palette does not leak outside the bezel.

```bash
git add "Conversational Builder v2.dc.html" screen-01.dc.html screen-02.dc.html
git commit -m "run: the app in its own palette, and the tool drawing over its edge"
```

---

### Task 6: `index.html` — two sections for screen 02's two states

**Files:**
- Modify: `index.html:104` (after screen 02's section, before `<footer>`)

**Interfaces:**
- Consumes: `screen-02.dc.html` from Tasks 2–5.
- Produces: nothing.

- [ ] **Step 1: Add sections 03 and 04**

The script picks up any `.embed` and reads `data-src`, so no JS change is needed. Insert after line 104:

```html
    <section class="piece" id="screen-03">
      <h2>
        <span class="num">03</span>
        <a class="titlelink" href="screen-02.dc.html" target="_blank" rel="noopener">Testing what exists, and finding the edge</a>
      </h2>
      <p class="desc">Two of five steps are built, so this run cannot finish — which makes the edge, not the preview, the subject. The check you asked to drop was built at 09:37, two minutes before you objected, so the run goes straight through it: unseen, three seconds, never in the customer's way. Then it stops, and the tool draws over the app to say so. Which is why <strong>Agree</strong> is here, on the step you have just watched run.</p>
      <div class="embed" data-src="screen-02.dc.html">
        <div class="fallback">Loading the run…</div>
      </div>
    </section>

    <section class="piece" id="screen-04">
      <h2>
        <span class="num">04</span>
        <a class="titlelink" href="screen-02.dc.html?at=1" target="_blank" rel="noopener">The same run, scrubbed back to the first step</a>
      </h2>
      <p class="desc">The rail is the scrubber, and the blue edge follows it — blue is where you are, and it has moved to the first row. Only the three steps that ran can be looked at; the last two cannot, because nothing of them exists to show. The phone's one live control is whatever the run's next move was, so the screen offers exactly what the run offered: one way on, or none.</p>
      <div class="embed" data-src="screen-02.dc.html?at=1">
        <div class="fallback">Loading the run…</div>
      </div>
    </section>
```

- [ ] **Step 2: Verify and commit**

Open `http://127.0.0.1:8765/index.html` and confirm four embeds, each scaled to the column, each clickable in place. Shoot it tall to check nothing is cut:

```bash
"$CHROME" --headless --disable-gpu --hide-scrollbars --force-device-scale-factor=1 \
  --window-size=1440,4200 --virtual-time-budget=8000 \
  --screenshot="$S/index.png" "http://127.0.0.1:8765/index.html"
```

```bash
git add index.html
git commit -m "docs: the run and its scrubbed-back state, framed in the index"
```

---

### Task 7: The rules the screen just added

**Files:**
- Modify: `CLAUDE.md` (the "Generated screens" section and "Two kinds of waiting")
- Modify: `docs/type-and-palette.md` (the palette section)

**Interfaces:**
- Consumes: everything above.
- Produces: nothing.

- [ ] **Step 1: `docs/type-and-palette.md` — two new rules**

Add after the blue "you are here" paragraph:

> **A previewed customer screen gets its own light palette, and it stays inside the bezel.**
> Ground `#ffffff`, surface `#f4f6f8`, text `#1a2129`, secondary `#5c6975`, hairline
> `#dfe5ea`, action `#2b7fe4`, radius 8 — the same type ladder and the same family. It is a
> different product, and the one confusion the test screen cannot afford is the app reading
> as another pane of the tool. When the tool needs to speak over the app it does so in the
> dark palette, which is what makes it legible as the tool: white is the thing you built.

> **The phone's bezel radius is hardware, not chrome.** 32px, outside the radius scale, the
> same way Phosphor's icon sizes sit outside the type ladder. It is the only off-scale
> radius in the repo, and it exists because a 360px slab at 12px does not read as a phone.
> There is still no shadow under it.

- [ ] **Step 2: `CLAUDE.md` — screen 02 in "Generated screens"**

The section already says "Add a screen to the source → re-run, then add its `index.html` section by hand." Nothing to change there. Add to "Two kinds of waiting", after the third bullet:

> - **A run of what exists may carry the wash for the step it is evidence about.** Screen
>   02's rail washes step 3 and carries `Agree` / `Comment further` because the run has just
>   demonstrated the claim under dispute — the check went out unseen in 3s, which is the
>   whole of *"it slows the customer down"*, answered. Wash it where the reply is given, and
>   the reply is best given here. Still one wash on that screen, still the same object the
>   plan washes, still the same words in all three views.

- [ ] **Step 3: Commit**

```bash
git add CLAUDE.md docs/type-and-palette.md
git commit -m "docs: a palette that stays inside the bezel, and a run that may wash its evidence"
```

---

## Self-review

**Spec coverage.** Geometry 360/440/640 → Task 2 Step 4. Chat turn + composer caption → Task 3. Rail header, rows, run lines, question mentions, group headings, trailing note → Task 4 Steps 1–3, 6, 7. Wash + act → Task 4 Step 4. Boundary → Task 4 Step 5. Inert rows 4–5 → Task 4 Step 6. Bezel, light palette, three states, caption, edge overlay, one-way-forward → Task 5. Top bar's five changes → Task 2 Step 3. Wiring (`state.at`, disjoint keys, no pins) → Task 1. `index.html` → Task 6. Docs → Task 7. Verification → the gate at the end of every task.

**Deliberately not done:** `Test what exists` on screen 01 and `Run again` here stay inert (spec, "Two deliberate inertnesses") — the screens are linked by `index.html`, and a `location.href` would jump you out of the source doc.

**Not applicable:** no `min-width` pins (nothing collapses), no `prefers-reduced-motion` work (the global rule already zeroes the hovers), no box-underrun trace (no animated measure on screen 02). The one-beat move is untouched, and the byte-identical screen 01 gate at every task is the proof.
