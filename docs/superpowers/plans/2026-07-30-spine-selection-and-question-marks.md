# Spine Selection + Question Marks Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Give the open step a blue "you are here" outline in the plan spine, and give every mention of an open question a grey `ph-question-mark`, per `docs/superpowers/specs/2026-07-30-spine-selection-and-question-marks-design.md`.

**Architecture:** All markup changes land in `Conversational Builder v2.dc.html` (the source doc — `screen-01.dc.html` is generated). The selection ring is a paint-only `outline` driven by three new `renderVals()` entries on the spine-label schedule. The glyphs are static `<i class="ph ph-question-mark">` insertions that inherit each count's own colour. Verification is the repo's own procedure: resting-state screenshot diffs (zero unexpected pixels) plus the box-underrun/judder probe from `docs/motion.md`.

**Tech Stack:** Static `.dc.html` rendered by `support.js`; headless Chrome for screenshots; Python 3 + Pillow (installed, v11.3.0) for pixel diffs.

## Global Constraints

- **Never `git push`.** Commits are fine; pushing publishes to a public URL (CLAUDE.md).
- Edit `Conversational Builder v2.dc.html` only — never `screen-01.dc.html`. Run `python3 build-screens.py` after every source edit.
- Preview over HTTP only (`python3 -m http.server 8765 --bind 127.0.0.1` from the repo root). Phosphor icons load from a CDN, so screenshots need network.
- The wash spec is untouchable: `linear-gradient(180deg, rgba(242,118,43,.14), rgba(242,118,43,.06) 45%, rgba(242,118,43,.025))` behind `1px solid rgba(242,118,43,.30)`. The ring goes *outside* it.
- `box-shadow` is only ever the 3px focus ring — selection must use `outline`.
- No new type sizes: glyphs are 10px, riding the existing 10/12 mono role.
- Glyph colour rule: the grey `ph-question-mark` takes the colour of the count/caption it precedes (`#8b98a5` or `#5c6975`), never a colour of its own. The orange one stays only on the `Not answered` card in the step pane.
- Repo root: `/Users/bogdandraghici/Desktop/vibes/AI Assisted Builder`. Scratchpad for baselines/diffs: `/private/tmp/claude-501/-Users-bogdandraghici-Desktop-vibes-AI-Assisted-Builder/3de4f9fc-2c02-4249-92d3-4af86cac9c14/scratchpad`.

---

### Task 1: Baselines + the blue selection outline

**Files:**
- Modify: `Conversational Builder v2.dc.html` (card div ~line 280; `renderVals()` after the `rmOpDl` entry ~line 1077)
- Regenerate: `screen-01.dc.html` via `python3 build-screens.py`

**Interfaces:**
- Produces: `renderVals()` keys `selOutline`, `selDur`, `selDl` (strings, CSS values); the spine card's `outline`/`outline-offset` styles. Task 3's probe assumes the card element is otherwise unchanged.

- [ ] **Step 1: Capture resting-state baselines from clean HEAD**

Confirm `git status` is clean first. Start the server if not already running, then:

```bash
cd "/Users/bogdandraghici/Desktop/vibes/AI Assisted Builder"
(python3 -m http.server 8765 --bind 127.0.0.1 >/dev/null 2>&1 &); sleep 1
SCRATCH="/private/tmp/claude-501/-Users-bogdandraghici-Desktop-vibes-AI-Assisted-Builder/3de4f9fc-2c02-4249-92d3-4af86cac9c14/scratchpad"
CHROME="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
"$CHROME" --headless --disable-gpu --window-size=1440,900 --hide-scrollbars --virtual-time-budget=6000 --screenshot="$SCRATCH/base-plan.png" "http://127.0.0.1:8765/screen-01.dc.html" 2>/dev/null
"$CHROME" --headless --disable-gpu --window-size=1440,900 --hide-scrollbars --virtual-time-budget=6000 --screenshot="$SCRATCH/base-step3.png" "http://127.0.0.1:8765/screen-01.dc.html?step=3" 2>/dev/null
```

Expected: two 1440×900 PNGs. Open both with the Read tool and confirm they rendered (chat pane, plan/spine, no blank page).

- [ ] **Step 2: Write the diff script**

Write to `$SCRATCH/diff.py`:

```python
import sys
from PIL import Image, ImageChops

a = Image.open(sys.argv[1]).convert('RGB')
b = Image.open(sys.argv[2]).convert('RGB')
assert a.size == b.size, (a.size, b.size)
d = ImageChops.difference(a, b)
bbox = d.getbbox()
if not bbox:
    print('IDENTICAL')
    sys.exit(0)
print('overall bbox:', bbox)
g = d.convert('L')
px = g.load()
w, h = d.size
rows = [y for y in range(h) if any(px[x, y] for x in range(w))]
bands = []
for y in rows:
    if bands and y - bands[-1][1] <= 3:
        bands[-1][1] = y
    else:
        bands.append([y, y])
for y0, y1 in bands:
    xs = [x for x in range(w) if any(px[x, y] for y in range(y0, y1 + 1))]
    print(f'band y={y0}-{y1} x={min(xs)}-{max(xs)}')
```

- [ ] **Step 3: Add the outline to the spine card**

In `Conversational Builder v2.dc.html`, find the washed card div (~line 280). Replace:

```html
              <div style="background: linear-gradient(180deg, rgba(242, 118, 43, 0.14), rgba(242, 118, 43, 0.06) 45%, rgba(242, 118, 43, 0.025)); border: 1px solid rgba(242, 118, 43, 0.30); border-radius: 12px; padding: {{ cardPad }}; display: flex; flex-direction: column; transition: padding {{ moveDur }} {{ moveEase }};">
```

with:

```html
              <!-- Selection, not status: the blue ring means "this is the step
                   open beside you" and nothing else. An outline, not a border,
                   so it is pure paint and no pinned measurement moves; the wash
                   keeps its own orange edge underneath, because selection and
                   "your move" are orthogonal and may stack. Same #4d97ea the
                   drawing's focal node wears. It arrives and leaves on the
                   spine labels' schedule, so the wide plan never flashes blue
                   and nothing new travels during the beat. -->
              <div style="background: linear-gradient(180deg, rgba(242, 118, 43, 0.14), rgba(242, 118, 43, 0.06) 45%, rgba(242, 118, 43, 0.025)); border: 1px solid rgba(242, 118, 43, 0.30); border-radius: 12px; padding: {{ cardPad }}; outline: 1px solid {{ selOutline }}; outline-offset: 3px; display: flex; flex-direction: column; transition: padding {{ moveDur }} {{ moveEase }}, outline-color {{ selDur }} {{ moveEase }} {{ selDl }};">
```

- [ ] **Step 4: Add the selection values to `renderVals()`**

Find the `rmOpDl: on ? '200ms' : '0ms',` line (~1077). Insert immediately after it:

```js

      // ── Selection: blue means "you are here" ─────────────────────────────
      // The open step's spine card wears a 1px #4d97ea outline. Selection is
      // structural, not a call to action, so it never spends orange — and it
      // only exists in the spine, where something IS open. Paint-only
      // (outline-color), on the spine labels' own schedule, for the same
      // reason theirs is: late in so the beat carries nothing new, fast out.
      selOutline: on ? 'rgba(77, 151, 234, 1)' : 'rgba(77, 151, 234, 0)',
      selDur: on ? '200ms' : '100ms',
      selDl: on ? '200ms' : '0ms',
```

- [ ] **Step 5: Rebuild the generated screen**

```bash
cd "/Users/bogdandraghici/Desktop/vibes/AI Assisted Builder" && python3 build-screens.py
```

Expected: exits 0, regenerates `screen-01.dc.html` (still carries the `GENERATED` banner).

- [ ] **Step 6: Screenshot both states and diff against baselines**

```bash
"$CHROME" --headless --disable-gpu --window-size=1440,900 --hide-scrollbars --virtual-time-budget=6000 --screenshot="$SCRATCH/t1-plan.png" "http://127.0.0.1:8765/screen-01.dc.html" 2>/dev/null
"$CHROME" --headless --disable-gpu --window-size=1440,900 --hide-scrollbars --virtual-time-budget=6000 --screenshot="$SCRATCH/t1-step3.png" "http://127.0.0.1:8765/screen-01.dc.html?step=3" 2>/dev/null
python3 "$SCRATCH/diff.py" "$SCRATCH/base-plan.png" "$SCRATCH/t1-plan.png"
python3 "$SCRATCH/diff.py" "$SCRATCH/base-step3.png" "$SCRATCH/t1-step3.png"
```

Expected:
- Wide plan: `IDENTICAL` — the outline is transparent there, so a change that touches one state must leave the other pixel-identical.
- Step-3 state: change bands **only** in a thin rectangular frame hugging the spine card (roughly x 365–655, y 295–415 — a ~1px ring 4px outside the card's edge). Any band elsewhere, or a thick band (content moved), is a failure.
- Read `t1-step3.png` and visually confirm: crisp 1px blue ring, visible gap to the orange border, clearly distinct from a focus ring.

- [ ] **Step 7: Commit**

```bash
cd "/Users/bogdandraghici/Desktop/vibes/AI Assisted Builder"
git add "Conversational Builder v2.dc.html" screen-01.dc.html
git commit -m "step: a blue ring for the step you have open — selection stops leaning on the wash

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 2: The grey question mark on every mention

**Files:**
- Modify: `Conversational Builder v2.dc.html` (eight insertions, listed per step)
- Regenerate: `screen-01.dc.html`

**Interfaces:**
- Consumes: nothing from Task 1 (independent edits; keep task order anyway so diffs stay attributable).
- Produces: static markup only — no new `renderVals()` keys.

The glyph is always `<i class="ph ph-question-mark" style="font-size: 10px;"></i>`, inheriting the count's colour. `ph-question-mark`, never `ph-question` (that's the top bar's help button).

- [ ] **Step 1: Rows 4 and 5, wide counts column**

Replace (exact, unique):

```html
<div style="font: 400 10px/12px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; color: #8b98a5;">2 to settle</div>
```

with:

```html
<div style="display: flex; align-items: center; gap: 4px; font: 400 10px/12px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; color: #8b98a5;"><i class="ph ph-question-mark" style="font-size: 10px;"></i>2 to settle</div>
```

Then the same for the `1 to settle` variant of that exact string (row 5, ~line 494 — the one **without** `padding-top` and **without** `background` in its style).

- [ ] **Step 2: Rows 4 and 5, spine under-title counts**

Replace:

```html
<div style="padding-top: 2px; font: 400 10px/12px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; color: #5c6975; white-space: nowrap;">2 to settle</div>
```

with:

```html
<div style="padding-top: 2px; display: flex; align-items: center; gap: 4px; font: 400 10px/12px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; color: #5c6975; white-space: nowrap;"><i class="ph ph-question-mark" style="font-size: 10px;"></i>2 to settle</div>
```

Then the same for its `1 to settle` twin (~line 489).

- [ ] **Step 3: The spine card's chip**

Replace (the chip whose text is `1 to settle` — its `2 not built` sibling stays bare, it is a build count, not a question):

```html
<div style="display: inline-flex; align-items: center; height: 18px; padding: 0 6px; border-radius: 4px; background: #131c24; border: 1px solid #26313d; color: #8b98a5; font: 400 10px/12px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;">1 to settle</div>
```

with:

```html
<div style="display: inline-flex; align-items: center; gap: 4px; height: 18px; padding: 0 6px; border-radius: 4px; background: #131c24; border: 1px solid #26313d; color: #8b98a5; font: 400 10px/12px ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;"><i class="ph ph-question-mark" style="font-size: 10px;"></i>1 to settle</div>
```

Also add this comment on the line above the `max-width: 206px` chip-row div that contains it (it documents the risk this task's Step 6 measures):

```html
                    <!-- A grey question mark marks a MENTION of a question; the
                         orange one marks the question itself, where you answer
                         it. Same rule as the wash: the glyph says which kind,
                         the colour only ever says "your move". It takes the
                         count's own colour, never one of its own. -->
```

- [ ] **Step 4: The plan subtitle**

In the subtitle div (~line 187), replace the text:

```
Two of five built · one not agreed · four to settle
```

with:

```html
Two of five built · one not agreed · <i class="ph ph-question-mark" style="font-size: 10px;"></i> four to settle
```

(Inline in flowing text, not flex — check baseline alignment visually in Step 6; if the glyph rides high, add `vertical-align: -1px` to its style.)

- [ ] **Step 5: The three captions**

The two identical footer captions (plan card ~line 416, step pane pinned footer ~line 871) — replace **both occurrences** (replace_all):

```html
<div style="font: 400 12px/16px 'Open Sans', sans-serif; color: #8b98a5; white-space: nowrap;">One question still to settle before I build it.</div>
```

with:

```html
<div style="display: inline-flex; align-items: center; gap: 4px; font: 400 12px/16px 'Open Sans', sans-serif; color: #8b98a5; white-space: nowrap;"><i class="ph ph-question-mark" style="font-size: 10px;"></i>One question still to settle before I build it.</div>
```

And the step pane's section note (~line 614):

```html
<div style="font: 400 12px/16px 'Open Sans', sans-serif; color: #8b98a5;">One question still to settle</div>
```

with:

```html
<div style="display: inline-flex; align-items: center; gap: 4px; font: 400 12px/16px 'Open Sans', sans-serif; color: #8b98a5;"><i class="ph ph-question-mark" style="font-size: 10px;"></i>One question still to settle</div>
```

The `Nothing to settle` note stays bare — zero questions is not a mention of one. The orange `ph-question-mark` on the `Not answered` card is untouched.

- [ ] **Step 6: Rebuild, screenshot, diff**

```bash
cd "/Users/bogdandraghici/Desktop/vibes/AI Assisted Builder" && python3 build-screens.py
"$CHROME" --headless --disable-gpu --window-size=1440,900 --hide-scrollbars --virtual-time-budget=6000 --screenshot="$SCRATCH/t2-plan.png" "http://127.0.0.1:8765/screen-01.dc.html" 2>/dev/null
"$CHROME" --headless --disable-gpu --window-size=1440,900 --hide-scrollbars --virtual-time-budget=6000 --screenshot="$SCRATCH/t2-step3.png" "http://127.0.0.1:8765/screen-01.dc.html?step=3" 2>/dev/null
python3 "$SCRATCH/diff.py" "$SCRATCH/t1-plan.png" "$SCRATCH/t2-plan.png"
python3 "$SCRATCH/diff.py" "$SCRATCH/t1-step3.png" "$SCRATCH/t2-step3.png"
```

Expected bands, and **nothing else** (a large or unexpected band means something rewrapped — stop and investigate):
- Wide plan: the subtitle line (~y 113–125); the card footer caption (right side, ~y 565–580); rows 4/5 count lines (right edge, ~y 655–735).
- Step-3 state: the chip (~y 380–392 in the spine card); the two spine under-title counts (~y 478–490 and ~y 540–552); the step pane section note (~y 370–382, right side) and pinned footer caption (~y 858–872).

Then Read both PNGs and verify with eyes:
1. **The chip pair `? 1 to settle` / `2 not built` still sits on ONE line** inside the spine card — this is the spec's named risk. If it wrapped: reduce both chips' `padding: 0 6px` to `0 5px` and re-check before touching any pinned number.
2. Rows 4/5: the counts column did not widen — "Nothing built yet ›" / "1 of 2 exists ›" must still be the widest line of their stacks (titles/descriptions beside them unchanged in the diff = proof).
3. The subtitle glyph sits on the baseline acceptably.

- [ ] **Step 7: Commit**

```bash
git add "Conversational Builder v2.dc.html" screen-01.dc.html
git commit -m "step: every mention of a question carries the question's mark, in grey

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 3: Prove the beat did not move

**Files:**
- Create (untracked, deleted at the end): `probe-underrun.html` in the repo root (must be served from the same origin as the screen).

**Interfaces:**
- Consumes: the built `screen-01.dc.html` from Tasks 1–2. No produced artifacts — this task is a gate.

- [ ] **Step 1: Write the probe**

Create `probe-underrun.html` in the repo root:

```html
<!DOCTYPE html>
<meta charset="utf-8">
<iframe id="f" src="screen-01.dc.html" style="width:1440px;height:900px;border:0"></iframe>
<pre id="out">running</pre>
<script>
const out = document.getElementById('out');
const sleep = ms => new Promise(r => setTimeout(r, ms));

async function hydrated(d) {
  for (let i = 0; i < 400; i++) {
    const busy = d.querySelector('.sc-placeholder') ||
                 d.documentElement.hasAttribute('sc-dc-streaming');
    const laid = [...d.querySelectorAll('div')]
      .some(el => el.getBoundingClientRect().top > 0);
    if (!busy && laid) return;
    await sleep(25);
  }
  throw new Error('never hydrated');
}

// The five step titles are the only divs whose inline min-height is in lh.
const titles = d => [...d.querySelectorAll('div')]
  .filter(el => /lh$/.test(el.style.minHeight));

function trace(d, anims, els) {
  let underrun = 0;
  const tops = els.map(() => []);
  for (let t = 0; t <= 480; t += 16.7) {
    anims.forEach(a => { try { a.currentTime = t; } catch (e) {} });
    els.forEach((el, i) => {
      const mh = parseFloat(getComputedStyle(el).minHeight) || 0;
      underrun = Math.max(underrun, el.scrollHeight - mh);
      tops[i].push(el.getBoundingClientRect().top);
    });
  }
  // Sum of steps against each row's net direction — totals, not worst step:
  // 19 frames of 1px the wrong way is the same judder as one frame of 19.
  const backTotals = tops.map(ts => {
    const net = ts[ts.length - 1] - ts[0];
    let s = 0;
    for (let i = 1; i < ts.length; i++) {
      const dlt = ts[i] - ts[i - 1];
      if (net > 0 ? dlt < 0 : dlt > 0) s += Math.abs(dlt);
    }
    return +s.toFixed(1);
  });
  return { underrun: +underrun.toFixed(1), backTotals };
}

async function move(d, els, open) {
  if (open) {
    // Opening is the card's own click. Clicking again would not close it —
    // the row's handler is openStep, and with the step open its
    // pointer-events are 'none' anyway; closing is the Esc listener.
    const cardTitle = [...d.querySelectorAll('div')]
      .find(el => el.textContent.trim().startsWith('We check with the merchant')
               && el.style.minHeight);
    cardTitle.dispatchEvent(new MouseEvent('click', { bubbles: true }));
  } else {
    d.dispatchEvent(new KeyboardEvent('keydown', { key: 'Escape', bubbles: true }));
  }
  await sleep(60);           // give it a turn, or getAnimations lies
  const anims = d.getAnimations();
  if (!anims.length) throw new Error('0 animations — clicked a hydrating page?');
  anims.forEach(a => a.pause());
  const r = trace(d, anims, els);
  anims.forEach(a => { try { a.finish(); } catch (e) {} });
  await sleep(60);
  return r;
}

(async () => {
  try {
    const d = document.getElementById('f').contentDocument;
    await hydrated(d);
    await d.fonts.ready;
    const els = titles(d);
    if (els.length !== 5) throw new Error('expected 5 titles, got ' + els.length);
    const opening = await move(d, els, true);    // plan -> step
    const closing = await move(d, els, false);   // step -> plan, via Esc
    out.textContent = 'RESULT ' + JSON.stringify({ opening, closing });
  } catch (e) {
    out.textContent = 'ERROR ' + e.message;
  }
})();
</script>
```

- [ ] **Step 2: Run it and read the numbers**

```bash
"$CHROME" --headless --disable-gpu --window-size=1500,1200 --virtual-time-budget=20000 --dump-dom "http://127.0.0.1:8765/probe-underrun.html" 2>/dev/null | grep -o 'RESULT {.*}\|ERROR .*'
```

Expected: a `RESULT` line. Acceptance (baselines from `docs/motion.md`: underrun 1.4/2.0, back-totals 2.2/0.0, interior rows ≤ 4.2):
- `underrun` ≤ 3px in each direction. Past 18px (a line) the wrap jumps are back — hard fail.
- every `backTotals` entry ≤ 5px in each direction.

If it fails: the chip row wrapping mid-flight is the prime suspect (spec's named risk) — check `t2-step3.png` chip line again, trim chip padding, rebuild, re-run. If the probe itself misbehaves (0 animations, flat trace), it clicked a hydrating page — raise the sleeps, per the two gates in `docs/motion.md`.

- [ ] **Step 3: Remove the probe**

```bash
rm "/Users/bogdandraghici/Desktop/vibes/AI Assisted Builder/probe-underrun.html"
```

No commit — this task changes nothing tracked.

---

### Task 4: The docs say what the screen now does

**Files:**
- Modify: `docs/type-and-palette.md` (end of Palette section)
- Modify: `CLAUDE.md` ("Two kinds of waiting" section)

**Interfaces:** none — prose only.

- [ ] **Step 1: `docs/type-and-palette.md`**

Append after the "**There are no left status stripes.**" paragraph:

```markdown

**Blue is "you are here"; it never means status.** The step open beside the spine wears a
1px solid `#4d97ea` outline at `outline-offset: 3px` — the same edge the drawing's focal
node wears. An outline, not a border, so it is pure paint and no pinned measurement
moves; and not a box-shadow, which stays reserved for the focus ring. The wash keeps its
own orange edge underneath, because selection and "your move" are orthogonal and may
stack. Selection exists only in the spine state — the wide plan never shows it, because
nothing is open there.

**Two question marks, one rule.** A grey `ph-question-mark` marks a *mention* of a
question — the "N to settle" counts, the chip, the captions — and takes the colour of
the count it precedes (`#8b98a5` or `#5c6975`), never one of its own. The orange one
marks the question itself, on the one card where you answer it, and nowhere else. Same
rule as the wash: the glyph says which kind, the colour only ever says "your move".
```

- [ ] **Step 2: `CLAUDE.md`**

In "Two kinds of waiting", the bullet beginning "**Never spend orange on a mention of a question**" currently ends "…which is a fact about the plan, not a call on you." Append two sibling bullets directly after that bullet:

```markdown
- **A mention still gets the question's glyph — in grey.** Every "N to settle" count and
  caption carries a `ph-question-mark` in the count's own grey, so the question kind of
  waiting is recognisable on the plan without spending orange. The orange glyph appears
  only on the answerable card in the step pane.
- **Selection never spends orange.** The step open beside the spine wears a 1px `#4d97ea`
  outline — blue is "you are here", orange is "your move", and they may stack. See
  [docs/type-and-palette.md](docs/type-and-palette.md).
```

- [ ] **Step 3: Commit**

```bash
git add docs/type-and-palette.md CLAUDE.md
git commit -m "docs: blue is where you are, orange is your move, and a grey mark for a mentioned question

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```
