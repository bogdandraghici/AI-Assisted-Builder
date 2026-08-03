# Notes

Measured numbers for the transition in `index.html`, and traps.
Everything else: read the page.

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
| Easing (settle beat) | `cubic-bezier(0.4, 0, 0.2, 1)`, **520ms** — its own |
| Settle beat | 520ms both ends; fade 360ms at +60 |
| Settle beat, blue drain | **480ms**, and it drains only — never fills |
| Settle beat, timers | p:1 → p:2 at **20ms**, unmount at `SETTLE + 40` |
| Jolt ceiling | **324px** of scroll clamp when applying from the bottom of the pane |
| Write field | **44px** (36 field + 8 gap); the dropped `because` returns **54** (48 + 6 gap) |

- Never `sc-if` anything that animates — unmounted, it has nothing to transition from.
- Collapse with `grid-template-rows: 1fr → 0fr` on a wrapper whose child is `min-height: 0`, and
  **clip the WRAPPER, not the child** — clipping the child leaves dead air, and `minmax(0, Xfr)`,
  `min-height: 0` and `align-content` do not fix it. The plan clips on the child by design.
- A collapsed wrapper still costs its column's `gap`. Cancel it on the same transition —
  `-12px` in the pane's lists, `-8px` inside `Answered by you`.
- Never nest two collapses: a `1fr` track inside a closed track resolves to nothing and does not
  recover. So the first answer into an empty list opens the LIST and is up itself from frame one;
  only a card landing in a list already up opens on its own.
- The settle beat is slower, softer and on its own easing because it is not a move. Folding
  ~350px out of a pane that scrolls: committed from the bottom, the held scroll stops existing
  and the browser clamps — 324px. A front-loaded curve puts a third of that in 100ms and reads
  as a reload; 520ms on a plain ease-out reads as a scroll.
- A transition needs a computed value to come from, so the beat's second half reads
  `document.body.offsetHeight` before it opens anything — the frame is forced, not hoped for.
- `max-width` must be smaller than the space at every instant: reserve the line box in `lh`,
  lead the measure opening, lag it closing, include flex `gap`s in the final.
- The pins are a function of the chat's width and the counts column — move either and all six
  are invalid. The fourth (979) is on the contested step's **card**.
- The plan does not scroll; anything added comes out of the 12px slack. The open-me arrow cost 4
  of the original 16 — its line box, on the four rows that had none.
- The write field under `Something else` costs 44 and the plan has 12, so the plan's copy drops
  the `because` (54) and slack goes 12 → **22**; the pane's copy keeps it and drops the drawing.
  The plan's wrapper is `overflow: hidden`, so an overrun clips 32px silently — measure underrun,
  not `scrollHeight`.
- The five rows are ONE `sc-for`, keyed by index (`support.js:639`): the list may never change
  length or order. Building a step changes its word; it does not move between `Built` and
  `Not built yet`.
- 194 is swept against `2lh` with **each** of the five as the open card, and the four that are
  not open take the same 194.

## Mechanics

- `style-hover`/`-active`/`-focus` reach `pseudoClass()` verbatim (`support.js:428`) and take
  **no** `{{ }}`. Every other attribute interpolates (`support.js:441`).
- Withdraw a control with `pointer-events` **and** `tabindex`, and blur on the act — a static
  `:focus` rule cannot be gated.
- Move focus in the `setState` callback, never in a `requestAnimationFrame`: an offscreen page
  is not painted, so the focus silently does not land.
- `Design System.dc.html` is the palette authority. One trap: warming the greys inside the wash
  goes mustard — flat grey, not a warmer grey.
- The plan carries **arguments** only; a detail keeps its card in its own step's pane. Three wash
  cards in that column is the plan overflowing, and it does not scroll.
- `Building` is grey. Orange is the wash, and a step being built is not waiting on you.
- The write field is `#0d0d0d` on `#2a2a2a` inside the wash, `#0e161d` on `#26313d` on the
  settled card, inset 24. Placeholders cannot be coloured inline — one `::placeholder` rule in
  the helmet, `#6b7783`, reads on both grounds.
- Two states, not one: `text` is the draft and dies with the card, `written` is the answer and
  outlives it. Both seed on open, the way `sel` seeds off `pick`.
- `Answered by you` reads in the order answers LANDED, and has to: the list is one `sc-for` keyed
  by index, so a card landing anywhere but the end renames the node already there. `order` is
  appended once per card.
- Applying keeps `sel` where you put it instead of clearing to `-1` — the card spends the next
  beat closing in front of you, and a row that de-selects on the way out is a flicker.
- The settled sentence reads through one function, so a card can never show settled on words
  nothing can find. A row with no `answer` is the field; that is the only test in the logic.

## Running it

```
python3 -m http.server 8765 --bind 127.0.0.1      # preview — always over HTTP
```

`file://` breaks `support.js` and the local fonts. No build: `index.html` is the screen, and the
states are `/`, `?step=3`, `?run=1`, `?run=1&at=1`, `?run=1&full=1`.

<https://github.com/bogdandraghici/AI-Assisted-Builder> is **public** — never push unless asked.
