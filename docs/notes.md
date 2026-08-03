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
| Easing (settle beat) | `cubic-bezier(0.4, 0, 0.2, 1)`, **520ms** — its own, and why is below |
| Settle beat | 520ms both ends; fade 360ms at +60 |
| Settle beat, blue drain | **480ms**, and it drains only — never fills |
| Settle beat, timers | p:1 → p:2 at **20ms**, unmount at **560ms** |

- Never `sc-if` anything that animates — unmounted, it has nothing to transition from.
- Collapse with `grid-template-rows: 1fr → 0fr` on a wrapper whose child is `min-height: 0`, and
  pin the collapsing content. **The clip goes on the WRAPPER, not the child.** A `<flex>` track is
  resolved twice: the container gets `f × H`, and then the track inside that now-definite height
  gets `f × f × H`. At `f = 0.5` the inner box is a quarter of the content while the wrapper still
  holds half — and if the inner box is the one clipping, the content is cut short of the space it
  occupies and the difference is dead air on the screen. `probe-fr` is five lines of that, measured.
  `minmax(0, Xfr)`, `min-height: 0` on the wrapper and `align-content` change nothing.
  The plan's own collapses still clip on the child; they were measured that way, and the plan clips
  by design. The settle beat is the one place the difference is visible, so it is the one that moved.
- A collapsed wrapper still costs its column's `gap`. Cancel it with a negative margin on the same
  transition — `-12px` in the pane's lists, `-8px` inside `Answered by you` — and the unmount at the
  end of the beat then costs no pixels.
- Never nest two collapses. A `1fr` track inside a track that is still closed resolves to nothing
  and does not recover when the outer one opens. So the first answer to land in an empty list lets
  the LIST open and is up itself from the first frame; only a card landing in a list already up
  opens on its own.
- The settle beat is slower and softer than the move, and on an easing of its own, because it is
  not a move. Folding ~350px out of a pane that scrolls drags everything above the card down to
  fill: at a normal scroll position anchoring holds the content still, but committed from the
  bottom of the pane the scroll it was holding stops existing and the browser clamps it — up to
  **324px** of travel, all of it honest. On the move's front-loaded curve a third of that lands in
  the first 100ms and reads as the page reloading. 520ms on a plain ease-out reads as a scroll.
  `probe-jolt` bounds it: where the scroll survives the beat the content may not move, and where
  it does not, the pane lands exactly on the clamp and never a pixel past it.
- A transition needs a computed value to come from. Two renders inside one frame leave a slot that
  mounted shut with nothing to open from, so the beat's second half reads `document.body.offsetHeight`
  before it opens anything — the frame is forced, not hoped for.
- `max-width` must be smaller than the space at every instant; reserve the line box in `lh`,
  leading the measure opening and lagging it closing. Include flex `gap`s in the final.
- The pins are a function of the chat's width and the counts column — move either and all six
  are invalid. The fourth (979) is on the contested step's **card**. A wider counts column is
  a motion change.
- The plan does not scroll; anything added to it comes out of the 12px slack. The open-me arrow
  on every row cost 4 of the original 16 — its line box, on the four rows that had none.
- The write field under `Something else` costs **44px** — 36 for the field, 8 for the band's gap
  — and the plan has 12. So the plan's copy of the card drops the `because` while you write,
  which is **54** (48 of text plus a 6px gap): slack goes 12 → **22**. The pane's copy keeps it
  and drops only the drawing. Overrunning here does not read as an overrun: the plan's own
  wrapper is `overflow: hidden`, so it clips 32px silently — measure underrun, not scrollHeight
  on the column.
- The five rows are ONE `sc-for`, keyed by index (`support.js:639`): the list may never change
  length or order, or the element has nothing to travel from. So building a step does not move
  it between `Built` and `Not built yet` — it changes its word.
- 194 is swept against `2lh` with **each** of the five as the open card, and the four that are
  not open get the same 194: below the space either way, and one measure instead of two.

## Measuring

Toggle, `document.getAnimations()`, `pause()` all, step every `currentTime` together in 16.7ms
increments, read rects out with `--dump-dom`. Probes: `probe-*.html`, gitignored — `probe-build`
settles every card and asserts Build arms at both grains, `probe-sweep` re-runs the `2lh` sweep
with each step open and reports the slack, `probe-write` drives the write field on both grounds
and asserts the plan neither overruns nor clips, `probe-doc` does the same to section 03 of
`Choice Cards`, `probe-settle` steps the settle beat and asserts both ends move and the column's
height is monotone through it, `probe-fr` is the five-case `<flex>` track reading above,
`probe-pose` stops the beat at one millisecond for `--screenshot`, `probe-jolt` rules out the two
ways the beat could look like a reload and bounds the third, `probe-shot` only poses a state for
`--screenshot`.

- `--virtual-time-budget` advances the clock in jumps of its own, so a real `wait` is not a way to
  land inside a transition: it can finish, and a finished transition is off `getAnimations()` and
  can no longer be posed. Step `currentTime` for the reading; for a screenshot, hold the beat's
  own timers instead (patch the iframe's `setTimeout` by delay) and never fire the unmount.
- A posed frame can paint stale: stepping an opacity transition by hand leaves the composited
  layer at its OLD size with its NEW alpha, which reads as a ghost overlapping what is below it.
  Hold the opacity at 0 (`probe-pose?nofade=1`) and the geometry paints as measured.

- Gate on hydration: `.sc-placeholder` gone, `sc-dc-streaming` off, tops non-zero. Fail loudly
  on zero geometry.
- Read state off the **inline** `style`, never computed. Colours come back as `rgb(...)`
  triples, not hex.
- To read a pin, lift every `min-width` to 0 at once and take the floor.
- Count line boxes with `createRange()` + `getClientRects()` collapsed by `top` — a reserved
  box hides its line count.
- `support.js` swallows a logic throw into an error placeholder and recovers, which on the screen
  is a flash and in the DOM is `.sc-placeholder-error` — so a probe that cares whether the page
  stayed up has to hook the PAGE's own `console.error` and `onerror`, not the harness's.
- Absolute judder figures are rig-specific; gate on a same-rig before/after. Box-underrun
  (~2px fine, 18px+ bad) is the one figure that reproduces.
- The browser folds `grid-row: 2; grid-column: 1` into `grid-area: 2 / 1`, so a probe that
  matches the written attribute finds nothing — read the resolved placement.
- The plan's filler is the **first** empty `flex: 1` div in the pane. Searching backwards
  finds the run strip's 2px connectors, which live in the same pane and read as 2px of slack.
- Matching a control by its label needs the DEEPEST match: when Build is live its gate
  sentence is gone, which leaves the act region's own text equal to the label.
- Headless stops painting once the page is offscreen and `requestAnimationFrame` stops with it,
  so a probe that ticks on rAF hangs after the first interaction — race it against a timeout.
- Typing into a controlled input from a probe needs the prototype `value` setter plus a bubbling
  `input` event; assigning `el.value` alone leaves React's state behind.
- The wide plan clips by design — every collapsed wrapper is a 0fr box with content in it — so a
  clip census only means something as a diff against the same page one interaction earlier.

## Mechanics

- `style-hover`/`-active`/`-focus` reach `pseudoClass()` verbatim (`support.js:428`) and take
  **no** `{{ }}`. Every other attribute interpolates (`support.js:441`).
- Withdraw a control with `pointer-events` **and** `tabindex`, and blur on the act — a static
  `:focus` rule cannot be gated.
- Move focus in the `setState` callback, never in a `requestAnimationFrame`: rAF waits on a
  paint, and an offscreen page is not painted, so the focus silently does not land.
- `Design System.dc.html` is the palette authority. One trap: warming the greys inside the
  wash goes mustard — flat grey is the answer, not a warmer grey.
- The plan carries **arguments** only. A detail keeps its card in its own step's pane — three
  wash cards in that column is the plan overflowing, and it does not scroll.
- `Building` is grey. Orange is the wash, and a step being built is not waiting on you.
- The write field is `#0d0d0d` on `#2a2a2a` inside the wash, `#0e161d` on `#26313d` on the
  settled card, inset 24 to hang under its row's label. A placeholder cannot be coloured inline
  — one `::placeholder` rule in the helmet, `#6b7783`, and it reads on both grounds.
- Two states, not one: `text` is the draft and dies with the card, `written` is the answer and
  outlives it. Both seed on open, the way `sel` seeds off `pick`.
- `Answered by you` reads in the order answers LANDED, not the step's card order, and it has to:
  the list is one `sc-for` keyed by index, so a card landing anywhere but the end renames the node
  that was already there and the node that opens is the wrong one. `order` is appended once per
  card — changing an answer you already gave is not a new one.
- Applying keeps `sel` where you put it instead of clearing it to `-1`. The card spends the next
  beat closing in front of you, and a row that de-selects itself on the way out is a flicker.
  Closed, nobody reads `sel`, and opening seeds it off `pick` — which is now the same row.
- The settled sentence reads through one function, so a card can never be shown settled on words
  nothing can find. A row with no `answer` is the field; that is the only test in the logic.

## Build

```
python3 build-screens.py                          # regenerate index.html
python3 -m http.server 8765 --bind 127.0.0.1      # preview — always over HTTP
```

`file://` breaks `support.js` and the local fonts. `index.html` is the screen itself, so the
states are URLs on it: `/`, `?step=3`, `?run=1`, `?run=1&at=1`, `?run=1&full=1`. A second
screen has nowhere to go — the build says so rather than guessing.

<https://github.com/bogdandraghici/AI-Assisted-Builder> is **public** — never push unless asked.
