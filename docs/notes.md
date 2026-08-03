# Notes

Every value is in `index.html`. This is only what the page cannot tell you: which numbers are
constrained by what, and what a wrong choice looked like.

- The six row `min-width` pins are a function of the chat's 360 and the counts column. Move
  either and all six are invalid; a wider counts column is a motion change. The fourth (979) is
  on the contested step's **card**, not its row.
- The settle beat's ceiling is **324px** of scroll clamp: committed from the bottom of a pane
  that scrolls, the held scroll stops existing and the browser clamps. A front-loaded curve puts
  a third of that in 100ms and reads as the page reloading — which is why the beat has 520ms and
  an easing of its own instead of the move's.
- The plan's 12px of slack is a budget. The open-me arrow cost 4 of the original 16, and the
  write field's 44 is why the plan's copy of the card drops the `because`. The plan's wrapper is
  `overflow: hidden`, so an overrun clips silently — measure the underrun, never `scrollHeight`.
- Never `sc-if` anything that animates: unmounted, it has nothing to transition from. And never
  nest two collapses — a `1fr` track inside a closed one resolves to nothing and does not recover.
- `style-hover`/`-active`/`-focus` reach `pseudoClass()` verbatim (`support.js:428`) and take
  **no** `{{ }}`. Every other attribute interpolates (`support.js:441`).
- Warming the greys inside the wash goes mustard. Flat grey is the answer.
