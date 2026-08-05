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
- The 360 chat's slack: start **14px**, mid-run ask **26**, mid-build **121**, everything else 70+.
  Buying the exchange's closing turn cost the 09:41 reply its second paragraph.
- The guided offer is one 28px chip, and in BOTH chats it is a sibling fold (`io` / `so`) between
  the history and the divider that replaces it — never a box inside a turn. A `1fr` inside a
  closed track resolves to nothing and never recovers, which is also why `bd`'s two inner folds
  carry `bd`'s own condition verbatim. Open, the chip cancels 20 of the column's 24 gap.
- The walk lives in the full-screen chat, so `guided` implies `intro` and is never a view. Its five
  blocks are twinned in the overlay and the 360 pane on ONE set of fold values — the only split is
  `igl`, because the 360 has to fold the lead on the first echo and the 640 has ~180px spare.
- The build block retires once the run it handed off to is up: five checks plus "the one thing left
  is to see it run" are **118px** the chat does not have, and the run answers both. The walk's
  promise (`rn`) folds the moment the build starts, for the same reason at **19px**.
- Nothing starts built. `?ask`, `?at`, `?live` and `?loading` are the only states that preset steps
  1 and 2 — a run of nothing is not a run, so every other entry point is withdrawn until something
  exists. The dot, the connector and each resource's existence are read off `built`, never written
  per step: `exists` names the step whose build makes it, and `by you` is the only pre-existing kind.
  The run states spill the frame by **24px**; that predates all of this.
- The guided ask is ONE block whose words swap only while the think beat holds it shut — its
  source row is a `display:` swap for the same reason, never `sc-if`.
- The intro is an overlay with `visibility`, never an `sc-if` around the body: the run screens'
  own `sc-if`s would nest untested, and the settle machinery must stay mounted underneath.
- The chat ⇄ plan swap is out **110** / in **200 at 110**, the chat travelling **32px** left and the
  grid **24px** right on the move curve. Three things it may not do: fade the 360 rail (the overlay's
  opaque dissolve uncovers it — a shared full-screen fade leaves ~50ms of blank screen, measured),
  fade the grid WITH the chat (the centred column stands over it, so they cross at half), or drop the
  body before the fade in ends — hence `bodyVisDl` **310**, the whole of it.
- A live run folds the plan-time exchange into its divider: open beside the ran note it is
  **122px** past the budget — that clip predates the intro and was invisible until the widget
  made Test what exists a first-screen path.
