# Notes

Every value is in `index.html`. This is only what the page cannot tell you: which numbers are
constrained by what, and what a wrong choice looked like.

- The six row `min-width` pins are a function of the chat's 360 and the counts column. Move
  either and all six are invalid; a wider counts column is a motion change. The fourth (979) is
  on the contested step's **card**, not its row.
- Above 1440 x 900 the shell **zooms** and the screen is laid out at the viewport divided by it, so
  the design is only ever larger and never re-proportioned; the axis that does not bind is slack and
  goes to the tracks. Two traps: `vw` / `vh` inside the zoom are z times too big, so `renderVals`
  hands `frameW` / `frameH` down instead — and at z = 1 the value must be `normal`, because a
  `zoom: 1` still takes the zoom path and re-rasterised every glyph in the run, 14.6k pixels off at
  the design size for a declaration meant to do nothing.
- The tracks are the region minus **321** (the band) or **300** (the spine), and
  every measure is capped: 1016 the plan's rows, 966 the app inside its frame, 906 the step pane.
  Each cap doubles as the floor of a pin at the design size, so the screen's own `min-width` /
  `min-height` may not move without all of them. A *height* pin may never be a percentage —
  against the cell that is collapsing it collapses too, and pins nothing.
- The settle beat's ceiling is **324px** of scroll clamp: committed from the bottom of a pane
  that scrolls, the held scroll stops existing and the browser clamps. A front-loaded curve puts
  a third of that in 100ms and reads as the page reloading — which is why the beat has 640ms and
  an easing of its own instead of the move's.
- The two ends of that beat may not share one fade. Faded together they cross at half, where the
  question is a ghost at full height and the answer is a ghost too, and the beat reads as a delete
  plus a jump. Out is **180 / 400** and in is **60 / 360** against the 640, so the answer is solid
  from ~420ms while the question still has a third of its height left — that overlap is the only
  thing on screen that says one became the other.
- The plan's 12px of slack is a budget. The open-me arrow cost 4 of the original 16, and the
  write field's 44 is why the plan's copy of the card drops the `because`. The plan's wrapper is
  `overflow: hidden`, so an overrun clips silently — measure the underrun, never `scrollHeight`.
- Never `sc-if` anything that animates: unmounted, it has nothing to transition from. And never
  nest two collapses — a `1fr` track inside a closed one resolves to nothing and does not recover.
- `style-hover`/`-active`/`-focus` reach `pseudoClass()` verbatim (`support.js:428`) and take
  **no** `{{ }}`. Every other attribute interpolates (`support.js:441`).
- Warming the greys inside the wash goes mustard. Flat grey is the answer.
