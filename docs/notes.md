# Notes

Measured values and traps for `Conversational Builder v2.dc.html`. **Reference, not a log** — keep
the number and the rule, drop the history. CLAUDE.md holds the principles; this holds the numbers.

---

## Motion — mechanism

CSS transitions over interpolated inline styles (`style="flex-basis: {{ planBasis }}"`);
`support.js` re-resolves them every render.

- **Never `sc-if` anything that animates** — unmounted, it has nothing to transition from.
- **One element per thing.** Wide plan and 300px spine are the *same* five rows, resized.
- **Collapse with `grid-template-rows: 1fr → 0fr`** (`-columns` horizontally) on a wrapper whose
  child is `overflow: hidden; min-height: 0`. Closed size is the content's own, so no magic number.
- **Fade before you close, fade after you open** (`exOp*` shorter than `ex*`). No stagger, no
  translate — the pane opening is the motion.
- **Pin collapsing content so it clips instead of reflowing.** A `1fr → 0fr` box's height is a
  fraction of its *content's*, so anything that rewraps grows while it should shrink.
- **Let the measure outrun the pane.** Title `max-width` 800 → 224 against space 879 → 224: on one
  curve `max-width` is always smaller, so it alone decides line breaks.
- **Reserve the line box early** — `min-height: 3lh`/`2lh` from `1lh`, leading the measure opening,
  lagging it closing. Rewrapping is 18px jumps and no curve smooths a jump.
- **What has no counterpart in the other state still has a size, and it is not late** — headings
  and counts only fade, but their 76px opens with everything else.

## Motion — measured values

| Thing | Value |
|---|---|
| Easing (move) | `cubic-bezier(0.2, 0, 0.2, 1)`, 340ms |
| Easing (hover) | same, 160ms |
| Easing (lead/lag: measure + reserved box) | `cubic-bezier(0, 0, 0.4, 0.8)` |
| `ldDur` closing | **240ms**, not 340 |
| Chat pane | fixed 360px, both states, takes no part in the move |
| Step pane content | `min-width: 780px` — uncovered, never squeezed |
| Row `min-width` pins @1440 | **838 / 838 / 860 / 979 / 860 / 860 / 1015** |
| Title `max-width` final (row) | `300 − 32 pad − 8 dot − 12 gap − 24 gap = 224` |
| Title `max-width` final (card) | `248 − 2 border − 20 pad − 24 gap − 8 gap = 194` |
| `tlA` / `tlB` reservations | 3 / 2 / 3 / 2 / 2 |
| Spine's real title line counts | 2 / 1 / 2 / 2 / 2 (under the reservation, deliberately) |
| `planPinH` | 816 / 812 |
| Box-underrun at rest | 1.4px opening, 2.0px closing |
| Full-screen frame | 1032 × 491 → 1392 × 804, chat and strip collapsing together |
| Full-screen pins | chat blocks 360, strip titles `3lh` (54px), boundary `2lh` (36px), underrun 0.0 |

- **The pins are a function of the chat's width and of what the counts column holds — move either
  and all seven are invalid.** The fourth (979) is on the contested step's **card**, not its
  contents: the card has an edge its contents would burst through first.
- **`max-width` finals must include gaps** — a collapsed sibling still occupies its flex `gap`.
- **Over-reserving is the safe direction.** Tightening the two loose reservations takes 36px out of
  the collapsed plan and gives the rows below that much further to travel in the same 340ms.
- **A wider counts column is a motion change.** It narrows the title at every instant, and a title
  narrower than its own animating `max-width` hands wrapping back to the pane.
- **Why `ldDur` closes at 240:** at 340 the panes rested ~300ms while 32px of reserved box was
  still collapsing, and rows drifted **19px back up** over the last 100ms. 240 puts the shrink
  inside the pane's own window — drift 19.4 → 1.4px, peak step 35 → 28px.
- **The stepped trace is stale**: the travelling rows moved to 237/241px when the contested card
  went under the row instead of around it. Expect ~27–31px/frame. Historic distance relation:
  24/26 over 196/200; 27/28 over 214/218; 24/31 over 190 at a 240px spine. Three beats instead of
  one peaked at 89px/frame with 5–7 reversals per row.
- **Absolute figures are rig-specific** — up to ~5.5px back-total on an *unchanged* build. Gate on
  a same-rig before/after. Box-underrun is the one figure that reproduces across rigs.

## Motion — how to measure

Click a probe's toggle, `document.getAnimations()`, `pause()` all, step every `currentTime`
together in 16.7ms increments, read `getBoundingClientRect().top` out with `--dump-dom`.

- **A sign change is worse than a big step, and total the backward deltas** — 19 frames of 1px is
  the same judder as one frame of 19, and only the total catches it (a worst-step-only summary
  called a 19px reversal 5.3px).
- **Box-underrun:** per frame, worst `scrollHeight − min-height` over the five titles. ~2px = no
  title gained a line. Past 18px the jumps are back.

### Traps

- **Gate on hydration with all three signals** — `.sc-placeholder` gone, `sc-dc-streaming` off,
  tops non-zero. Placeholders go *before* bindings resolve, so a two-signal probe reads raw markup
  with every rect zero and prints PASS. That produced a confident, wrong report that `support.js`
  does not interpolate `tabindex`/`title`/`class`.
- Check the **target's own** `style` for `{{`, never `document.body.innerHTML` — it holds the
  probe's braces. And **fail loudly on impossible geometry**: PASS over zero rects is worse than
  no check.
- **Give the click a turn to render** before `getAnimations()`, or you get zero animations, a flat
  lying trace, and labels one click behind.
- **Measure with webfonts loaded** (`document.fonts.ready` + explicit `fonts.load()`). The tell is
  rows with no counts column staying stable while the others move.
- **To read a pin, lift every `min-width` to 0 at once** — a pin can only report itself — and take
  the **floor**: a pin above the real width overflows its clipping parent.
- **A reserved line box hides its line count** (`scrollHeight` can't go below the element's own
  height). Count line boxes: `createRange()`, `selectNodeContents`, collapse `getClientRects()` by
  `top`. A clone with the floor lifted lands on the wrong side of a borderline wrap at 206/224px.
- **Headless Chrome:** `--virtual-time-budget` advances layout but not compositor opacity, and
  stepped traces are impossible there. **Check both ends** instead: transitions off, toggle each
  resting state, `scrollHeight − offsetHeight` over every pinned box. No box smaller than its
  content at either end ⇒ none can gain a line between.
- **Forcing a state: replace the whole initialiser expression.** Swapping only
  `get('step') === '3'` leaves `location.search).true` — legal JS evaluating to `undefined`, so the
  page renders the state you were leaving and the test passes for the wrong reason.

### The gate for a copy change

Per resting state: every description one line at its own `min-width`, every title one line at 800
in the wide plan, no pinned box with a non-zero underrun, `planPinH` still 816 / 812. Then
screenshot-diff: zero differing pixels, or bands one line-height tall where the copy changed. A
taller band, or one below your last edit, is layout shifting. One state changed ⇒ the other stays
pixel-identical.

---

## Rules with a value attached

CLAUDE.md states the principles. These are the specifics that are easy to get wrong.

- **Negotiation** is `Not agreed` → `Agreed`, glyph `ph-warning`, settled in the step's footer.
  **A missing detail** is `N to settle` / `Not answered`, glyph `ph-question-mark` (*not*
  `ph-question`, the top bar's help), answered in place. Counted in both views in their own units.
- **An argument's two acts are agree and comment further** — *build* is the agent's move and
  belongs in no button here. The act row lives outside the scroller with the ×; if it grows, the
  scroller's `bottom` changes with it.
- **Two objects may carry the wash:** the argument about the contested step, and the one open
  question in the step pane. Two washes on screen is right. A run may carry it for the step it is
  evidence about — wash it where the reply is given.
- **The wash is on the argument, not the step.** Wide plan: ordinary row, washed card *under* it.
  Spine: that card clips to nothing, so the wash falls back onto the step's block and carries its
  own `Not agreed` eyebrow (`rmRows`/`rmOp`), wrapped with the title in a `gap: 0` box — a 0fr grid
  still pays its column's 2px gap and would drop that one title 2px below its dot.
- **`Built` / `Not built yet`, no third heading**; the run strip says `Ran` instead, the one
  deliberate difference between views, because a run can only claim what *exists*.
- **The gutter answers its heading's question only.** Step 3's dot is row 4's dashed grey dot; in
  the run strip it is a green check like steps 1–2. `ph-warning` survives on the step pane's
  eyebrow alone.
- **One settle tag per plan row**, the whole right-hand column, all five rows.
- **Orange is rationed to** the settle tags, the wash, and step 3's eyebrow — not the group
  headings, not the gutter. The contested step's chips stay grey in the spine, where its block is
  already washed; in the wide plan that row has no wash, so its tag is orange like the rest.
- **The header's clauses do not partition the five steps** — build, argument, questions are three
  axes and step 3 sits on all three.

### Withdrawing a control

Both states are always mounted, so a control that cannot act now is withdrawn in the logic. (Not
the same as unimplemented chrome — the bell, kebab and undo pair hover and do nothing.)

- **Gate with `pointer-events` in the interpolated `style`.** `style-hover`/`-active`/`-focus`
  reach `pseudoClass()` verbatim (`support.js:428`) and take **no** `{{ }}`.
- **Withdraw the `tabindex` with it**, and never leave a pointer target the keyboard cannot reach.
  A clipped pane does both at once via `visibility` (inherits, discrete, cannot judder), hidden on
  a delay equal to the content's fade-out.
- **`{{ }}` resolves in every attribute** (`support.js:441`) — `tabindex`, `title`, `class` like
  `style`, the `style-*` trio excepted. A state-dependent glyph is one element with an interpolated
  attribute, never two behind an `sc-if`.
- **Three glyphs:** `caret-right` discloses in this pane, `arrow-right` changes what you look at,
  `arrows-out`/`-in` changes how much of it. A control and its row carry the same one.
- **Four ways out of a step are one control at four sizes** — ×, Esc, breadcrumb, back arrow share
  `exitTab` / `upPE`; all quiet in the plan state.

---

## Type, spacing and palette

`Design System.dc.html` is the authority; this is what it comes to here.

- **Ladder** (§04): nine roles over 10 · 12 · 14 · 18 · 24, line-heights 12 · 16 · 18 · 22 · 24 ·
  28 · 38. **No 11px, no 13px.** **Spacing:** 4 · 6 · 8 · 12 · 16 · 24 · 56, off-scale only for
  derived optical alignment.
- **Two families, load-bearing.** Open Sans is plan language; mono is what the product says *about*
  the plan — identifiers, counts, timestamps, micro-labels (`10/12 · 400` at `+.14em`), chips.
  Prose caps at 800px.
- **Palette** (§01, §03): ground `#0b1218`, side pane `#0e161d`, lifted card `#131c24`, outlined
  card `#101820` on `#1c2530`, hairline `#18212a`.
- **The wash** is `linear-gradient(180deg, rgba(242,118,43,.14), rgba(242,118,43,.06) 45%,
  rgba(242,118,43,.025))` behind `1px solid rgba(242,118,43,.30)` — always 180°, thinning but never
  out. The settle tag's flat tint at the same values is a different object: `rgba(242,118,43,.14)`
  on `rgba(242,118,43,.30)` when owed, `#131c24` on `#26313d` when not.
- **No elevation shadows**; `box-shadow` is the 3px focus ring only. A control on the wash hovers
  to `#131c24`, never opening a hole to `#0b1218`.
- **Blue is "you are here", never status:** `1px solid #4d97ea` at `outline-offset: 3px` — an
  outline, so no pinned measurement moves. Spine state only.
- **`ph-question-mark` has no colour of its own** — it takes the colour of what it precedes.
- **The previewed customer screen is light and never leaves its frame:** `#ffffff`, surface
  `#f4f6f8`, text `#1a2129`, secondary `#5c6975`, hairline `#dfe5ea`, action `#2b7fe4`, radius 8 —
  same ladder, same family, its own layout (two columns, 24/38 where a phone stacks at 18/28). The
  frame is chrome: 12px radius, no shadow. The tool always speaks in the dark palette.

---

## Build and preview

```
python3 build-screens.py                          # regenerate screen-NN.dc.html
python3 -m http.server 8765 --bind 127.0.0.1      # preview — always over HTTP
```

`file://` breaks `support.js` and the local fonts. Open Sans ships from `fonts/`; Phosphor icons
come from a CDN, so icons need network.

- **Screens are generated.** `build-screens.py` splits each top-level `data-screen-label` block out
  of the source doc. Edit the source, never `screen-*.dc.html` — overwritten, `GENERATED` banner,
  swept when removed. New screen → re-run, then add its `index.html` section by hand.
- **One page, five states** — one block, embedded once per state off `location.search` (`?step=3`,
  `?run=1`, `?run=1&at=1`, `?run=1&full=1`). Two blocks drift; that is how the chat panes
  misaligned once.
- **Publishing.** <https://github.com/bogdandraghici/AI-Assisted-Builder> is **public**, branch
  `main`; Pages serves the root ~20s after a push — hence the explicit-ask rule in CLAUDE.md.
