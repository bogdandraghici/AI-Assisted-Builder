# Notes

Measured values and traps for `Conversational Builder v2.dc.html`. Reference, not a log — keep the
number and the rule, drop the history. CLAUDE.md holds the principles.

---

## Motion — mechanism

CSS transitions over interpolated inline styles (`style="flex-basis: {{ planBasis }}"`);
`support.js` re-resolves them every render.

- Never `sc-if` anything that animates — unmounted, it has nothing to transition from.
- One element per thing. Wide plan and 300px spine are the *same* five rows, resized.
- Collapse with `grid-template-rows: 1fr → 0fr` (`-columns` horizontally) on a wrapper whose child
  is `overflow: hidden; min-height: 0`. Closed size is the content's own.
- Fade before you close, fade after you open (`exOp*` shorter than `ex*`). No stagger, no translate.
- Pin collapsing content so it clips instead of reflowing — a `1fr → 0fr` box's height is a fraction
  of its *content's*, so anything that rewraps grows while it should shrink.
- Let the measure outrun the pane: `max-width` smaller than the space at every instant, so it alone
  decides line breaks.
- Reserve the line box early (`min-height` in `lh`), leading the measure opening, lagging it closing.
  Rewrapping is 18px jumps and no curve smooths a jump.
- What has no counterpart in the other state still has a size, and it is not late.

## Motion — measured values

| Thing | Value |
|---|---|
| Easing (move) | `cubic-bezier(0.2, 0, 0.2, 1)`, 340ms |
| Easing (hover) | same, 160ms |
| Easing (lead/lag: measure + reserved box) | `cubic-bezier(0, 0, 0.4, 0.8)` |
| `ldDur` closing | **240ms**, not 340 |
| Chat pane | fixed 360px, both states, takes no part in the move |
| Step pane content | `min-width: 780px` |
| Row `min-width` pins @1440 | **838 / 838 / 860 / 979 / 860 / 860 / 1015** |
| Title `max-width` final (row) | `300 − 32 pad − 8 dot − 12 gap − 24 gap = 224` |
| Title `max-width` final (card) | `248 − 2 border − 20 pad − 24 gap − 8 gap = 194` |
| `titleLines` reservation | 2lh, one value for all five (`1lh` in the wide plan) |
| Spine's real title line counts | 2 / 1 / 2 / 2 / 2 |
| `planPinH` | 816 / 812 — *available* height, viewport less `planPad` |
| Box-underrun at rest | 1.4px opening, 2.0px closing |
| Mid-close reserved-box shortfall | 2.9px row 3, 1.3px row 4 — a few px for ~50ms, not a line |
| Full-screen frame | 1032 × 491 → 1392 × 804, chat and strip collapsing together |
| Full-screen pins | chat 360, strip titles `3lh`, boundary `2lh`, underrun 0.0 |

- The pins are a function of the chat's width and of what the counts column holds — move either and
  all seven are invalid. The fourth (979) is on the contested step's **card**, not its contents.
- `max-width` finals must include gaps — a collapsed sibling still occupies its flex `gap`.
- A wider counts column is a motion change: it narrows the title at every instant, and a title
  narrower than its own animating `max-width` hands wrapping back to the pane.
- `ldDur` closes at 240 because at 340 the panes rested ~300ms while the reserved boxes were still
  collapsing, and rows drifted 19px back up. Drift 19.4 → 1.4px, peak step 35 → 28px.
- Reservations: tighten only against a sweep. Over-reserving costs a visible hole wherever something
  sits under the title; under-reserving costs a jump.
- Expect ~27–31px/frame at 237/241px of travel. Absolute figures are rig-specific (~5.5px back-total
  on an unchanged build) — gate on a same-rig before/after. Box-underrun reproduces across rigs.

## Motion — how to measure

Click a probe's toggle, `document.getAnimations()`, `pause()` all, step every `currentTime` together
in 16.7ms increments, read `getBoundingClientRect().top` out with `--dump-dom`.

- A sign change is worse than a big step — total the backward deltas. 19 frames of 1px is the same
  judder as one frame of 19.
- Box-underrun: per frame, worst `scrollHeight − min-height` over the five titles. ~2px = no title
  gained a line; past 18px the jumps are back.
- To sweep a reservation without animating: rebuild the curves in JS, set the real element's
  `max-width`/`font-size`/`line-height` to each sampled instant, lift `min-height` to 0, count line
  boxes. Both directions — the shortfalls live at the start of the close. `lh` moves with the
  animating line-height, so report shortfall as a fraction of a line.

### Traps

- Gate on hydration with all three signals — `.sc-placeholder` gone, `sc-dc-streaming` off, tops
  non-zero. Placeholders go *before* bindings resolve, so a two-signal probe reads raw markup with
  every rect zero and prints PASS.
- Check the **target's own** `style` for `{{`, never `document.body.innerHTML`. Fail loudly on
  impossible geometry: PASS over zero rects is worse than no check.
- Give the click a turn to render before `getAnimations()`.
- Measure with webfonts loaded (`document.fonts.ready` + explicit `fonts.load()`).
- To read a pin, lift every `min-width` to 0 at once — a pin can only report itself — and take the
  floor.
- `planPinH` is not a content height: the block is stretched by its flex parent, so `min-height: 0`
  does not shrink it. Take a content change's cost off the rows below.
- A reserved line box hides its line count. Count line boxes with `createRange()` +
  `selectNodeContents`, collapsing `getClientRects()` by `top`, on the element itself — a clone
  lands on the wrong side of a borderline wrap.
- Headless Chrome cannot step compositor frames. Check both ends instead: transitions off, toggle
  each resting state, `scrollHeight − offsetHeight` over every pinned box.
- Forcing a state: replace the whole initialiser expression, or you get legal JS evaluating to
  `undefined` and a test that passes for the wrong reason.

### The gate for a copy change

Per resting state: every description one line at its own `min-width`, every title one line at 800 in
the wide plan, no pinned box with a non-zero underrun, `planPinH` still 816 / 812. Then
screenshot-diff: zero differing pixels, or bands one line-height tall where the copy changed. One
state changed ⇒ the other stays pixel-identical.

---

## Rules with a value attached

- **Negotiation** is `Not agreed` → `Agreed`, glyph `ph-warning`, settled in the step's footer. **A
  missing detail** is `N to settle` / `Not answered`, glyph `ph-question-mark` (*not* `ph-question`,
  the top bar's help), answered in place. Counted in both views in their own units.
- An argument's two acts are agree and comment further — *build* is the agent's move. The act row
  lives outside the scroller with the ×.
- Wash it where the reply is given, and nowhere the act is withdrawn. Wide plan: ordinary row, washed
  card under it. Spine: that card clips to nothing and the wash goes with it — the acts are inside
  the clip. One wash on screen per state.
- What crosses into the spine is the word: the `Not agreed` eyebrow (`rmRows`/`rmOp`), wrapped with
  the title in a `gap: 0` box — a 0fr grid still pays its column's 2px gap. Without it the spine
  names no contested step at all; blue means *you are here*, not status.
- `Built` / `Not built yet`, no third heading; the run strip says `Ran` instead, because a run can
  only claim what *exists*.
- The gutter answers its heading's question only. Step 3's dot is row 4's dashed grey dot.
  `ph-warning` survives on the step pane's eyebrow alone.
- One settle tag per plan row, the whole right-hand column, all five.
- Orange is rationed to the settle tags, the wash, and step 3's eyebrow. Every count is grey in the
  spine, all five rows, bare 10/12 mono under its own title at `padding-top: 2px`; in the wide plan
  all five are orange pills.
- The header's clauses do not partition the five steps — build, argument, questions are three axes
  and step 3 sits on all three.

### Withdrawing a control

Both states are always mounted, so a control that cannot act now is withdrawn in the logic. (Not the
same as unimplemented chrome — the bell, kebab and undo pair hover and do nothing.)

- Gate with `pointer-events` in the interpolated `style`. `style-hover`/`-active`/`-focus` reach
  `pseudoClass()` verbatim (`support.js:428`) and take **no** `{{ }}`.
- Withdraw the focus ring too, by blurring on the act — `style-focus` is a static `:focus` rule, so
  a control that withdraws itself keeps a ring nothing can dismiss. `leave(patch)` blurs
  `currentTarget`; Escape blurs `activeElement`. Not `toggleFull` — it stays live in both states.
- Withdraw the `tabindex` with it, and never leave a pointer target the keyboard cannot reach. A
  clipped pane does both via `visibility` (inherits, discrete, cannot judder).
- `{{ }}` resolves in every attribute (`support.js:441`) — the `style-*` trio excepted. A
  state-dependent glyph is one element with an interpolated attribute.
- Three glyphs: `caret-right` discloses in this pane, `arrow-right` changes what you look at,
  `arrows-out`/`-in` changes how much of it.
- Four ways out of a step are one control at four sizes — ×, Esc, breadcrumb, back arrow share
  `exitTab` / `upPE`.

---

## Type, spacing and palette

`Design System.dc.html` is the authority; this is what it comes to here.

- **Ladder** (§04): nine roles over 10 · 12 · 14 · 18 · 24, line-heights 12 · 16 · 18 · 22 · 24 ·
  28 · 38. No 11px, no 13px. **Spacing:** 4 · 6 · 8 · 12 · 16 · 24 · 56.
- Two families, load-bearing. Open Sans is plan language; mono is what the product says *about* the
  plan — identifiers, counts, timestamps, micro-labels (`10/12 · 400` at `+.14em`), chips. Prose
  caps at 800px.
- **Palette** (§01, §03): ground `#0b1218`, side pane `#0e161d`, lifted card `#131c24`, outlined
  card `#101820` on `#1c2530`, hairline `#18212a`.
- **The wash:** `linear-gradient(180deg, rgba(242,118,43,.14), rgba(242,118,43,.06) 45%,
  rgba(242,118,43,.025))` behind `1px solid rgba(242,118,43,.30)`, always 180°. The settle tag is a
  flat tint at the same values: `.14` on `.30` when owed, `#131c24` on `#26313d` when not.
- No elevation shadows; `box-shadow` is the 3px focus ring only. A control on the wash hovers to
  `#131c24`.
- Blue is "you are here", never status: `1px solid #4d97ea` at `outline-offset: 3px`, an outline so
  no pinned measurement moves. Spine state only.
- `ph-question-mark` takes the colour of what it precedes.
- The previewed customer screen is light and never leaves its frame: `#ffffff`, surface `#f4f6f8`,
  text `#1a2129`, secondary `#5c6975`, hairline `#dfe5ea`, action `#2b7fe4`, radius 8 — same ladder,
  same family, two columns at 24/38 where a phone stacks at 18/28. Frame: 12px radius, no shadow.

---

## Build and preview

```
python3 build-screens.py                          # regenerate screen-NN.dc.html
python3 -m http.server 8765 --bind 127.0.0.1      # preview — always over HTTP
```

`file://` breaks `support.js` and the local fonts. Open Sans ships from `fonts/`; Phosphor icons
come from a CDN, so icons need network.

- Screens are generated. `build-screens.py` splits each top-level `data-screen-label` block out of
  the source doc. Edit the source, never `screen-*.dc.html`. New screen → re-run, then add its
  `index.html` section by hand.
- One page, five states — one block, embedded once per state off `location.search` (`?step=3`,
  `?run=1`, `?run=1&at=1`, `?run=1&full=1`). Two blocks drift.
- <https://github.com/bogdandraghici/AI-Assisted-Builder> is **public**, branch `main`; Pages serves
  the root ~20s after a push — hence the explicit-ask rule in CLAUDE.md.
