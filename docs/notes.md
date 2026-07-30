# Notes

Numbers that cost real measurement, for `Conversational Builder v2.dc.html`. Nothing here that a
look at the screen would tell you.

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
| Plan slack @1440×900 | **9px** — the filler at the end of the plan column |
| Title `max-width` final (row) | `300 − 32 pad − 8 dot − 12 gap − 24 gap = 224` |
| Title `max-width` final (card) | `248 − 2 border − 20 pad − 24 gap − 8 gap = 194` |
| `titleLines` | 2lh, one value for all five (`1lh` wide) |
| Spine's real title line counts | 2 / 1 / 2 / 2 / 2 |
| `planPinH` | 816 / 812 — *available* height, viewport less `planPad` |
| Box-underrun at rest | 1.4px opening, 2.0px closing |
| Mid-close reserved-box shortfall | 2.9px row 3, 1.3px row 4 |
| Travel | ~27–31px/frame over 237/241px |
| Full-screen frame | 1032 × 491 → 1392 × 804 |
| Full-screen pins | chat 360, strip titles `3lh`, boundary `2lh`, underrun 0.0 |

- Never `sc-if` anything that animates — unmounted, it has nothing to transition from.
- Collapse with `grid-template-rows: 1fr → 0fr` on a wrapper whose child is `overflow: hidden;
  min-height: 0`. Pin the collapsing content: a `1fr → 0fr` box's height is a fraction of its
  *content's*, so anything that rewraps grows while it should shrink.
- `max-width` must be smaller than the space at every instant, so it alone decides line breaks.
  Reserve the line box in `lh`, leading the measure opening and lagging it closing.
- The pins are a function of the chat's width and of what the counts column holds — move either and
  all six are invalid. The fourth (979) is on the contested step's **card**, not its contents.
- The plan does not scroll, and the contested card holds the step pane's whole card. Anything added
  to that card comes out of 9px — measure the plan column's trailing filler before and after.
- `max-width` finals must include gaps — a collapsed sibling still occupies its flex `gap`.
- A wider counts column is a motion change: it narrows the title at every instant.
- Absolute figures are rig-specific (~5.5px back-total on an unchanged build). Gate on a same-rig
  before/after. Box-underrun is the one figure that reproduces.

### Measuring

Toggle, `document.getAnimations()`, `pause()` all, step every `currentTime` together in 16.7ms
increments, read `getBoundingClientRect().top` out with `--dump-dom`.

- Total the backward deltas — 19 frames of 1px is the same judder as one frame of 19.
- Box-underrun per frame: worst `scrollHeight − min-height` over the five titles. ~2px = no title
  gained a line; past 18px the jumps are back.
- To sweep a reservation without animating: rebuild the curves in JS, set the real element's
  `max-width`/`font-size`/`line-height` to each sampled instant, lift `min-height` to 0, count line
  boxes. Both directions — shortfalls live at the start of the close.
- `probe-scroll.html?at=1` parks the step pane's scroller at the bottom — a still cannot reach it,
  and the pane is the only scroller on the screen.
- Gate on hydration with all three signals: `.sc-placeholder` gone, `sc-dc-streaming` off, tops
  non-zero. Fail loudly on zero geometry — PASS over zero rects is worse than no check.
- Measure with webfonts loaded. Give a click a turn to render. Check the target's own `style` for
  `{{`, not `document.body.innerHTML`.
- To read a pin, lift every `min-width` to 0 at once and take the floor.
- A reserved line box hides its line count — count line boxes with `createRange()` +
  `selectNodeContents`, collapsing `getClientRects()` by `top`, on the element itself.
- `planPinH` is not a content height: the block is stretched by its flex parent, so `min-height: 0`
  does not shrink it.
- Headless Chrome cannot step compositor frames. Check both ends: transitions off, toggle each
  resting state, `scrollHeight − offsetHeight` over every pinned box.
- Forcing a state: replace the whole initialiser expression, or you get legal JS evaluating to
  `undefined` and a test that passes for the wrong reason.

**Gate for a copy change:** per resting state, every description one line at its own `min-width`,
every title one line at 800 wide, no pinned box underrunning, `planPinH` still 816 / 812. Then
screenshot-diff — bands one line-height tall where the copy changed, and the other state
pixel-identical.

## Mechanics worth knowing

- `style-hover`/`-active`/`-focus` reach `pseudoClass()` verbatim (`support.js:428`) and take **no**
  `{{ }}`. Every other attribute interpolates (`support.js:441`).
- Withdraw a control with `pointer-events` **and** `tabindex` in the interpolated style, and blur on
  the act — a static `:focus` rule cannot be gated, so a control that withdraws itself keeps a ring
  nothing can dismiss. `leave(patch)` blurs `currentTarget`; Escape blurs `activeElement`.
- A clipped pane withdraws both at once via `visibility` — inherits, discrete, cannot judder.
- Glyphs: `ph-warning` for negotiation, `ph-question-mark` for a missing detail (*not*
  `ph-question`, the top bar's help). `caret-right` discloses here, `arrow-right` changes what you
  look at, `arrows-out`/`-in` how much of it.

## Palette

`Design System.dc.html` is the authority. Ground `#0b1218`, side pane `#0e161d`, lifted card
`#131c24`, outlined card `#101820` on `#1c2530`, hairline `#18212a`. Selection `#4d97ea` as an
outline at `offset: 3px`. `box-shadow` is the 3px focus ring only, never elevation.

The wash: `linear-gradient(180deg, rgba(242,118,43,.14), rgba(242,118,43,.06) 45%,
rgba(242,118,43,.025))` behind `1px solid rgba(242,118,43,.30)`. The settle tag is a flat tint at
the same values — `.14` on `.30` when owed, `#131c24` on `#26313d` when not.

Inside the wash the cool ladder goes warm-neutral, saturation ≤10% — ground `#161413`, lifted
`#1f1c1b`, outlined `#1a1817` on `#2d2a28`, hairline `#262322`, line-strong `#3b3735`. Wireframe
insets sink to `#0e0d0c` and draw at `#4c4745`. Past ~20% saturation these go mustard; the hue is
not the problem, the chroma is. Blue stays cool and means taken, so hover lifts the ground and
never takes the border.

Ladder: 10 · 12 · 14 · 18 · 24 over line-heights 12 · 16 · 18 · 22 · 24 · 28 · 38. No 11px, no
13px. Spacing 4 · 6 · 8 · 12 · 16 · 24 · 56. Prose caps at 800px. Open Sans is plan language, mono
is what the product says *about* the plan (micro-labels `10/12 · 400` at `+.14em`).

The previewed customer screen is light: `#ffffff`, surface `#f4f6f8`, text `#1a2129`, secondary
`#5c6975`, hairline `#dfe5ea`, action `#2b7fe4`, radius 8; two columns at 24/38 where a phone
stacks at 18/28. Frame 12px radius, no shadow.

## Build

```
python3 build-screens.py                          # regenerate screen-NN.dc.html
python3 -m http.server 8765 --bind 127.0.0.1      # preview — always over HTTP
```

`file://` breaks `support.js` and the local fonts. Phosphor icons come from a CDN.

One page, five states off `location.search` (`?step=3`, `?run=1`, `?run=1&at=1`, `?run=1&full=1`),
embedded once per state. Two blocks drift. New screen → re-run the build, then add its
`index.html` section by hand.

<https://github.com/bogdandraghici/AI-Assisted-Builder> is **public** — never push unless asked.
