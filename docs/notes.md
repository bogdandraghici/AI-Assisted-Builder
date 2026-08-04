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
