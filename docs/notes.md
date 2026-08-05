# Notes

Measured numbers and traps only. Every value lives in `index.html`.

- The six row `min-width` pins are a function of the chat's 360 and the counts column; move
  either and all six are invalid. The fourth (979) is on the contested step's **card**, not its row.
- Above 1440 x 900 the shell zooms and the screen is laid out at the viewport divided by z.
  `vw` / `vh` inside the zoom are z times too big — `renderVals` hands down `frameW` / `frameH`.
  At z = 1 the value must be `normal`: a `zoom: 1` still takes the zoom path and re-rasterises
  every glyph, 14.6k pixels off at the design size.
- Tracks are the region minus **321** (band) or **248** (spine). Caps: 1016 plan rows, 966 app,
  906 step pane — each doubles as a pin's floor, so the screen's `min-width` / `min-height` may
  not move without all of them. A height pin may never be a percentage.
- The spine's 248 is derived, not chosen: **170** is the tightest title measure all five still
  wrap to two lines in (165 puts one on three), plus 74 of chrome, plus 4 of slack. A collapsed
  column still leaves its flex gap — the chip's 24 and the caret's 8 fold to 0 in the spine.
- Settle beat: **640ms**, `cubic-bezier(0.65, 0, 0.35, 1)`, symmetric. Ceiling is **324px** of
  scroll clamp. Any ease-out is a trap at this size — `0.4, 0, 0.2, 1` put nine tenths of a 336px
  fold in the first 375ms. Measure the fold in the browser before believing a curve.
- The flyer is `cloneNode` of the card's own node, so there is nothing to crossfade. Its own
  border comes off, and the settled ground washes in **over** the clone, never under.
- The flyer closes its OFFSET from the landing card, never the gap to it: the slot is still
  opening, and interpolating towards the live rect overshoots by 130px.
- Every duration in the beat is **`0s` off the beat**. `sc-for` keys by index (`support.js:639`),
  so the unmounting card hands its node to whatever was under it; on a duration the survivor
  animated back open — **670k pixels in one frame**, 32% of the screen. Sorting the settling card
  last only moves the jump to the front of the beat.
- The fades for the paths with no flyer: out at **180 / 400**, in at **60 / 360**. They may not
  share one — faded together they cross at half, both ghosts.
- Diff consecutive **screencast** frames (`Page.startScreencast`, not `screenshot` — 80ms apart
  misses a one-frame flash). Every jump found here was invisible to `getBoundingClientRect`.
- The plan has **12px** of slack. The open-me arrow cost 4 of the original 16; the write field's
  44 is why the plan's card drops the `because`. The wrapper is `overflow: hidden`, so an overrun
  clips silently — measure the underrun, never `scrollHeight`.
- Never `sc-if` anything that animates, and never nest two collapses: a `1fr` track inside a
  closed one resolves to nothing and does not recover.
- `style-hover`/`-active`/`-focus` reach `pseudoClass()` verbatim (`support.js:428`) and take
  **no** `{{ }}`. Every other attribute interpolates (`support.js:441`).
- Warming the greys inside the wash goes mustard. Flat grey.
- The 640 column scrolls, the 360 rail does not. Two states overrun it: the interjection mid-walk
  with the exchange still open and the lead still up (**74px**, and the argument's second row was
  under the composer), and the walk's build block (**105**). It sticks to the bottom on every frame
  of the beat, not at the commit — the block that overruns is a fold, and at the commit its height
  is still zero. Bar hidden both ways (`scrollbar-width` and `::-webkit-scrollbar`): the column
  travels to the rail's measure and a bar that takes width lands it a scrollbar off. Leaving folds
  the overrunning blocks on the rail's condition, so the scroll clamps to 0 before the travel.
- The 360 chat's slack: start **14px**, mid-run ask **26**, mid-build **121**, everything else 70+.
  Buying the exchange's closing turn cost the 09:41 reply its second paragraph.
- The guided offer is one 28px chip, and in BOTH chats it is a sibling fold (`io` / `so`) between
  the history and the divider that replaces it — never a box inside a turn. A `1fr` inside a
  closed track resolves to nothing and never recovers, which is also why `bd`'s two inner folds
  carry `bd`'s own condition verbatim. Open, the chip cancels 20 of the column's 24 gap.
- The walk lives in the full-screen chat, so `guided` implies `intro` and is never a view. Its five
  blocks are twinned in the overlay and the 360 pane on ONE set of fold values — the only split is
  `igl`, because the 360 has to fold the lead on the first echo and the 640 has ~180px spare. Every
  split holds only while the chat is UP: on the way out they take the rail's condition, or the
  column has nothing to land on.
- The build block retires once the run it handed off to is up: five checks plus "the one thing left
  is to see it run" are **118px** the chat does not have, and the run answers both. The walk's
  promise (`rn`) folds the moment the build starts, for the same reason at **19px**.
- Nothing starts built. `?ask`, `?at`, `?live` and `?loading` are the only states that preset steps
  1 and 2 — a run of nothing is not a run, so every other entry point is withdrawn until something
  exists. The dot, the connector and each resource's existence are read off `built`, never written
  per step: `exists` names the step whose build makes it, and `by you` is the only pre-existing kind.
  The run states spill the frame by **24px**; that predates all of this.
- The guided ask is ONE block whose words swap only while the think beat holds it shut — its
  source row is a `display:` swap for the same reason, never `sc-if`. Shut is not instant: the
  block reads `gShow`, holding the answered card for the beat, because swapping on the commit the
  fold starts draws the NEXT question at full opacity for **two frames** (~50ms, screencast) before
  the 70ms fade catches up. Invisible with two text rows either side, a jump with a 96px drawing.
- The intro is an overlay with `visibility`, never an `sc-if` around the body: the run screens'
  own `sc-if`s would nest untested, and the settle machinery must stay mounted underneath.
- The chat ⇄ plan swap is the column MOVING onto the rail, not a dissolve: 640 → **311** wide,
  `50%`/`-320px` → `0%`/`24px`, padding-top 48 → 24, over **320ms** of the settle curve, and only
  then a **140ms** crossfade — a fade over the last of the travel is the same sentence in two places
  30px apart. It lands to the pixel: measure **277** both sides (311 − 22 avatar − 12 gap), and the
  composer block is **104** both once its bottom padding grows to 32 — 24 of padding, 8 of gap, and
  a caption that is 16px of nothing until a run is live (48 when there is one).
  Four rules hold it: the 360 rail is never animated (the overlay's opaque dissolve uncovers it — a
  shared full-screen fade leaves ~50ms of blank screen, measured); the grid may not share that
  dissolve and arrives at **320** instead; the offset may never be read off the viewport, because a
  resize retargets it every frame and a transition whose target keeps moving restarts its delay and
  never lands; and anything that does NOT land is handed over instead — the ask block and the
  composer's line are typeset for 640 and the rail's copies for 311, and `igl` / `lq` / `lr` / `ld`
  fold on the RAIL's own condition the moment the chat is down, a block the rail folds being 72 or
  170px the landing does not have. Diff the two trees before touching any of it.
- A live run folds the plan-time exchange into its divider: open beside the ran note it is
  **122px** past the budget — that clip predates the intro and was invisible until the widget
  made Test what exists a first-screen path.
