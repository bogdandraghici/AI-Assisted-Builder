# AI Assisted Builder

Design-doc prototypes for the FlowX AI-assisted builder. Pages are `.dc.html`: static
HTML rendered client-side by `support.js`, which supplies `<x-dc>`, `sc-for`,
`style-hover` / `style-active` / `style-focus`, `DCLogic` and React.

## Every page ships in two variants

1. **Standalone** — the page on its own, filling the viewport. No caption, heading,
   description or card framing. What it would look like in the real product.
2. **Embedded** — framed inside `index.html` at its design size and scaled to the
   column. The index owns the title and description, so the standalone page never
   repeats them.

Build any new page with both variants in mind.

## Generated screens

Standalone screens are generated, not hand-written:

```
python3 build-screens.py
```

It splits each top-level `data-screen-label` block out of
`Conversational Builder v2.dc.html` into `screen-NN.dc.html`, stripping the doc
framing (caption, black canvas, padding, 1440×900 card with border and radius).

- Edit the source doc, never `screen-*.dc.html` — they are overwritten on every run
  and carry a `GENERATED` banner.
- The script hard-fails with a clear message if the source is restyled such that it
  can no longer strip the framing, rather than emitting a broken page.
- Add a screen to the source → re-run, then add its `index.html` section by hand.
- Remove a screen from the source → re-run and its page is deleted. Only pages
  carrying the banner are swept, so a hand-written `screen-*.dc.html` is safe.

## States, not screens

A screen with two states is **one** `data-screen-label` block, not two. The plan and
the plan-with-a-step-open are the same page; `index.html` shows both by embedding it
twice, the second time as `screen-01.dc.html?step=3`, which the `Component` reads off
`location.search`. Two blocks would mean two copies of the same markup, and they
drift — that is how the chat panes ended up misaligned once already.

Motion is CSS transitions over state-driven inline styles — `style="flex-basis: {{ planBasis }}"`,
with `renderVals()` returning the value for each state. `support.js` re-resolves style
strings on every render, so this animates; three rules follow from it:

- **Never `sc-if` anything that animates.** It unmounts, and an unmounted element has
  no value to transition from. `sc-if` is for things that just appear, like the
  technical-detail table.
- **One element per thing, never two that cross-fade.** The wide plan and the 240px
  spine are the *same* five rows changing type size, dot size and padding. Two layers
  dissolving into each other was the first attempt and it read as broken: you saw the
  wide plan clipped mid-sentence while a differently-shaped rail ghosted in under it.
  If the two states share a thing, it must be one element.
- **Collapse with `grid-template-rows: 1fr → 0fr`** (and `-columns` horizontally) on a
  wrapper whose child is `overflow: hidden; min-height: 0`. The closed size is then
  exactly the content's own height, with no magic number to keep in sync as copy
  changes — which `max-height` needs.
- **Fade before you close, fade after you open.** A column closing on live text leaves
  a one-character sliver, and that single frame is what makes a move look cheap. The
  `exOp*` values run shorter than the `ex*` ones for exactly this.

The move is **one** beat. The panes resize while the detail collapses while the spine's
own labels open their space while the padding tightens — one curve, one duration, both
directions, everything starting at the same instant. It was three beats first, then two,
and both read exactly as the complaint each time: "it happens in two steps", then "there
is too much going on". A sequence is not something to schedule well; it is something to
remove, and two specific things force one if you let them.

- **Reflowing content cannot collapse while the pane narrows over it — so stop it
  reflowing.** A `1fr → 0fr` box's height is a fraction of its *content's* height, so a
  description that rewraps mid-collapse makes that fraction grow while it is meant to be
  shrinking, and every row below it lurches. That is the whole reason the collapse used
  to have to finish before the panes could start. Give each collapsing block a `min-width`
  equal to the width it has when open and it clips instead of reflowing — invisibly, since
  it is at zero opacity long before the pane reaches it. Measure the number, don't guess
  it: they differ per row (876 / 876 / 945 / 848 / 875 / 1015 at 1440) because the counts
  column beside each one does. They are a function of the chat's width, so anything that
  moves that pane invalidates all six — they all shifted 80px when the chat settled on one
  360 width. And measure with the webfonts *loaded*: gate the probe on `document.fonts.ready`
  plus an explicit `fonts.load()` of the weights the plan uses, or a late-arriving face
  shifts every counts column a few px and the numbers wobble run to run.
- **Rewrapping is not motion, it is a stack of 18px jumps** — one per line a title gains —
  and no curve smooths a jump. The old answer was to hide them inside a collapse violent
  enough to cover them, which is *why* the collapse was violent: 60px in a frame. The real
  answer is to reserve the line box before the text needs it — `min-height: 3lh` / `2lh`
  animating from `1lh`, leading the measure, so the wrap happens inside a box that is
  already big enough. Five titles restacking then move nothing at all. That one change
  took the interior rows from 15–25px lurches to a 4px-a-frame drift and let the collapse
  slow down to 20px a frame. Closing, it must *lag* the measure instead — the box may only
  shrink once the text has un-wrapped, or the content sets the height again and the jumps
  come back. The `lh` unit keeps it honest: if copy ever makes a count wrong, that title
  degrades to a jump, not to a broken layout.
- **Let the measure outrun the pane and the pane never touches the wrapping.** Both start
  together, but a title's `max-width` travels 800 → 164 while the space available to it
  travels 879 → 164, so on the same curve the max-width is *always* the smaller of the two
  and it alone decides where lines break. That is what makes a simultaneous move legal at
  all. It must equal the true final width, gaps included — a collapsed sibling still
  occupies its flex `gap`, so the first attempt was 24px too generous and bought one last
  rewrap on the final frame. Derive it: row `240 - 32 pad - 8 dot - 12 gap - 24 gap = 164`;
  card `164 - 20 pad - 2 border - 12 gap - 8 gap = 146`. It applies to anything that wraps,
  not just titles: the pair of chips in the card broke onto a second line 20ms before the
  dock landed and threw the two rows below it down 26px in one frame.
- **Whatever has no counterpart in the other state still has a SIZE, and the size is not
  late.** The Agreed / You are here / Next headings and the per-step counts can only fade,
  but the 76px they occupy belongs to the shape the plan is collapsing into, so that space
  opens with everything else and only the opacity waits. Given a beat of its own it made
  five rows fly up past where they belonged and slide a hundred pixels back down once the
  rest had settled.
- **Arrivals do not need a stagger to feel considered.** The step's five blocks each slid
  up 10px in their own 40ms wave, which is five more things moving while the plan is
  collapsing and the panes are resizing. One fade, no translate: the pane opening is the
  motion, and the content is what it uncovers.

To check smoothness, don't look — measure. Click the toggle in a probe script, then
`document.getAnimations()`, `pause()` them all, and step every animation's `currentTime`
together in 16.7ms increments, reading `getBoundingClientRect().top` of each row into an
attribute you can pull out with `--dump-dom`. That gives an exact per-frame trace with no
dependence on wall time. Then diff consecutive frames: a **sign change is worse than a
big step** — 60px in one frame is a fast collapse, 20px the wrong way is judder. Gate on
the probe having actually rendered (`.sc-placeholder` gone, `sc-dc-streaming` off, tops
non-zero), or you will click a hydrating page, get zeros, and read them as real.

Where it stands, at 16.7ms per frame: the two rows that travel 190px peak at 24px a frame
opening and 31px closing, monotonic apart from a single sub-3px settle; the interior rows
never step more than 4px in either direction. The three-beat version peaked at 89px a
frame with 5–7 reversals per row and was still moving at 700ms. It is 340ms now.

The resting states are the check that the machinery is invisible: screenshot both, diff
against the previous build, and expect **zero** differing pixels. `min-height: 3lh` must
equal the spine's real line count and `min-width` the open width, so if either is wrong
the diff says so immediately.

Easing is the FlowX `cubic-bezier(0.2, 0, 0.2, 1)` at 340ms for the move and 160ms for
hovers. The two things that lead or lag it — the measure and the reserved line box — use
`cubic-bezier(0, 0, 0.4, 0.8)` to get ahead early. The step pane still clips rather than
reflows: its content holds a `min-width: 838px` so an opening pane uncovers it instead of
squeezing it — the same trick as the pinned collapsing blocks, and it was there first.

Chrome's `--virtual-time-budget` advances layout transitions but **not** compositor
opacity, so mid-flight frames captured that way lie — they showed an empty middle that
was actually a 47%-opacity spine. To check a transition, render a static frame with
hand-solved intermediate values. And when patching the logic to force a state for such
a frame, replace the *whole* initialiser expression: swapping only `get('step') === '3'`
leaves `location.search).true`, which is legal JS evaluating to `undefined`, so the page
quietly renders the state you were trying to leave and the test passes for the wrong
reason.

Design size is 1440×900. Standalone pages fill the viewport but hold a
`min-width` / `min-height` at that size — below it the panes and table columns
collide, so the page scrolls instead of breaking.

## Type and spacing

`Design System.dc.html` §04 holds the type ladder — nine named roles over five sizes
(10 · 12 · 14 · 18 · 24) and only FlowX line-heights (12 · 16 · 18 · 22 · 24 · 28 · 38).
Pick a role; never invent a size. **There is no 11px and no 13px** — both existed before
and made the levels indistinguishable, which is the whole problem the ladder solves.
Spacing is the FlowX scale (4 · 6 · 8 · 12 · 16 · 24 · 56); an off-scale number is only
allowed when it is derived optical alignment, not spacing — e.g. `padding-left: 28px`
to sit under a 12px number gutter plus its 16px gap.

Two families, and the distinction is load-bearing: Open Sans is plan language; mono is
everything the product says *about* the plan — identifiers, counts, timestamps, and
every uppercase micro-label and state chip. A micro-label is `10/12 · 400 mono` at
`+.14em`, and it labels a panel or a column and nothing else. Prose inside the elastic
column is capped at 800px so a line stays readable.

Palette is §01 and §03 — ground `#0b1218`, side pane `#0e161d`, lifted card `#131c24`,
outlined card `#101820` behind `#1c2530`, hairline `#18212a`. **There are no elevation
shadows.** The one exception per screen is carried by a wash of the disagreement orange
plus a tinted edge — always 180°, strongest at the top edge and thinning, but never all
the way out, by the bottom — and
settled things take a 3px left status stripe instead. `box-shadow` is only ever the 3px
focus ring.

## Preview

```
python3 -m http.server 8765 --bind 127.0.0.1
```

Always over HTTP. Opening `file://` breaks `support.js` and the local fonts.
Open Sans ships from `fonts/`; Phosphor icons come from a CDN, so icons need network.

## Git

- Repo: <https://github.com/bogdandraghici/AI-Assisted-Builder> — **public**, branch `main`
- Pages: served from `main` at the repo root →
  <https://bogdandraghici.github.io/AI-Assisted-Builder/>, redeploying on every push
  (~20s). `.nojekyll` keeps files served verbatim.

**Never push unless I explicitly ask.** Commit freely — commits are local and
reversible — but a push publishes to a public URL that gets indexed. Prefer `git revert`
over rewriting history; no force-pushing what is already published.
