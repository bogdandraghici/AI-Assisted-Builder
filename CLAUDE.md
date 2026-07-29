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

Motion is CSS transitions over state-driven inline styles — `style="width: {{ chatW }}"`,
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

The move is three beats, and the order is the whole trick: the detail collapses at full
width, *then* the panes resize, *then* the labels only the spine carries fade in. Doing
it in one pass means five descriptions and a diagram all reflowing while the column
shrinks under them.

Anything that genuinely has no counterpart in the other state — the Agreed / You are
here / Next headings, the per-step counts — arrives last and leaves first, because it
cannot morph from anything.

Easing is the FlowX `cubic-bezier(0.2, 0, 0.2, 1)` at 360ms for panes and 160ms for
hovers. The step pane still clips rather than reflows: its content holds a
`min-width: 838px` so an opening pane uncovers it instead of squeezing it.

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
