# Notes

Measured numbers for the transition in `Conversational Builder v2.dc.html`, and traps.
Everything else: read the source doc.

## Motion

CSS transitions over interpolated inline styles; `support.js` re-resolves them every render.

| Thing | Value |
|---|---|
| Easing (move) | `cubic-bezier(0.2, 0, 0.2, 1)`, 340ms |
| Easing (hover) | same, 160ms |
| Easing (lead/lag: measure + reserved box) | `cubic-bezier(0, 0, 0.4, 0.8)` |
| `ldDur` closing | **240ms**, not 340 |
| Chat pane | fixed 360px, takes no part in the move |
| Step pane content | `min-width: 780px` |
| Row `min-width` pins @1440 | **838 / 838 / 860 / 979 / 860 / 860** |
| Plan slack @1440×900 | **12px** — the filler at the end of the plan column |
| Title `max-width` final | **194**, one value for all five rows — gaps included |
| `titleLines` | 2lh, one value for all five |
| `planPinH` | 816 / 812 — *available* height, viewport less `planPad` |
| Full-screen pins | chat 360, strip titles `3lh`, boundary `2lh` |

- Never `sc-if` anything that animates — unmounted, it has nothing to transition from.
- Collapse with `grid-template-rows: 1fr → 0fr` on a wrapper whose child is `overflow: hidden;
  min-height: 0`, and pin the collapsing content.
- `max-width` must be smaller than the space at every instant; reserve the line box in `lh`,
  leading the measure opening and lagging it closing. Include flex `gap`s in the final.
- The pins are a function of the chat's width and the counts column — move either and all six
  are invalid. The fourth (979) is on the contested step's **card**. A wider counts column is
  a motion change.
- The plan does not scroll; anything added to it comes out of the 12px slack. The open-me arrow
  on every row cost 4 of the original 16 — its line box, on the four rows that had none.
- The five rows are ONE `sc-for`, keyed by index (`support.js:639`): the list may never change
  length or order, or the element has nothing to travel from. So building a step does not move
  it between `Built` and `Not built yet` — it changes its word.
- 194 is swept against `2lh` with **each** of the five as the open card, and the four that are
  not open get the same 194: below the space either way, and one measure instead of two.

## Measuring

Toggle, `document.getAnimations()`, `pause()` all, step every `currentTime` together in 16.7ms
increments, read rects out with `--dump-dom`. Probes: `probe-*.html`, gitignored — `probe-build`
settles every card and asserts Build arms at both grains, `probe-sweep` re-runs the `2lh` sweep
with each step open and reports the slack.

- Gate on hydration: `.sc-placeholder` gone, `sc-dc-streaming` off, tops non-zero. Fail loudly
  on zero geometry.
- Read state off the **inline** `style`, never computed. Colours come back as `rgb(...)`
  triples, not hex.
- To read a pin, lift every `min-width` to 0 at once and take the floor.
- Count line boxes with `createRange()` + `getClientRects()` collapsed by `top` — a reserved
  box hides its line count.
- Absolute judder figures are rig-specific; gate on a same-rig before/after. Box-underrun
  (~2px fine, 18px+ bad) is the one figure that reproduces.
- The browser folds `grid-row: 2; grid-column: 1` into `grid-area: 2 / 1`, so a probe that
  matches the written attribute finds nothing — read the resolved placement.
- The plan's filler is the **first** empty `flex: 1` div in the pane. Searching backwards
  finds the run strip's 2px connectors, which live in the same pane and read as 2px of slack.
- Matching a control by its label needs the DEEPEST match: when Build is live its gate
  sentence is gone, which leaves the act region's own text equal to the label.

## Mechanics

- `style-hover`/`-active`/`-focus` reach `pseudoClass()` verbatim (`support.js:428`) and take
  **no** `{{ }}`. Every other attribute interpolates (`support.js:441`).
- Withdraw a control with `pointer-events` **and** `tabindex`, and blur on the act — a static
  `:focus` rule cannot be gated.
- `Design System.dc.html` is the palette authority. One trap: warming the greys inside the
  wash goes mustard — flat grey is the answer, not a warmer grey.
- The plan carries **arguments** only. A detail keeps its card in its own step's pane — three
  wash cards in that column is the plan overflowing, and it does not scroll.
- `Building` is grey. Orange is the wash, and a step being built is not waiting on you.

## Build

```
python3 build-screens.py                          # regenerate screen-NN.dc.html
python3 -m http.server 8765 --bind 127.0.0.1      # preview — always over HTTP
```

`file://` breaks `support.js` and the local fonts. New screen → re-run the build, then add its
`index.html` section by hand.

<https://github.com/bogdandraghici/AI-Assisted-Builder> is **public** — never push unless asked.
