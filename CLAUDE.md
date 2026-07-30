# AI Assisted Builder

Design-doc prototypes for the FlowX AI-assisted builder. Pages are `.dc.html`: static
HTML rendered client-side by `support.js`, which supplies `<x-dc>`, `sc-for`,
`style-hover` / `style-active` / `style-focus`, `DCLogic` and React.

## This file is principles only

Every number, mechanism, measurement and worked example lives in `docs/`. The detail there is
load-bearing and was mostly paid for by getting it wrong once — **read the doc before changing
anything it covers**, and when you learn something new, put it in the doc rather than here.

- **[docs/motion.md](docs/motion.md)** — the one-beat move: why it is one beat, the five things
  that make one beat possible, the measured `min-width` / `min-height` pins, the easing and
  durations, how to prove a change did not judder, and the traps in headless Chrome.
- **[docs/affordances.md](docs/affordances.md)** — how a control is withdrawn when it cannot
  act, what `{{ }}` does and does not interpolate, the three glyphs, the status words.
- **[docs/waiting.md](docs/waiting.md)** — the two kinds of waiting, what the orange wash
  means, what may carry it, and where the act sits.
- **[docs/type-and-palette.md](docs/type-and-palette.md)** — the type ladder, the spacing
  scale, the two families, the palette.
- **[docs/build-and-preview.md](docs/build-and-preview.md)** — the standalone/embedded pair,
  `build-screens.py`, how one page carries two states, how to serve it all.

## Principles

### Every page ships in two variants

**Standalone** is the page on its own filling the viewport, with no doc framing: what it would
look like in the real product. **Embedded** is that same page framed inside `index.html` at its
design size, where the index owns the title and description — so the standalone page never
repeats them. Build any new page with both in mind. Design size is 1440×900.

### Screens are generated, never hand-written

`python3 build-screens.py` splits the source doc into `screen-NN.dc.html`. Edit the source
doc; a generated page is overwritten on the next run.

### States, not screens

A screen with two states is **one** `data-screen-label` block, not two, shown twice by
`index.html`. Two blocks would be two copies of the same markup, and they drift.

### The two states are joined by one beat, and it is the most delicate thing here

Three of its rules are easy to break without opening [docs/motion.md](docs/motion.md):

- **Never `sc-if` anything that animates** — an unmounted element has no value to transition
  from. `sc-if` is for things that just appear.
- **One element per thing, never two that cross-fade.** The wide plan and the 300px spine are
  the same five rows changing size, not two layers dissolving.
- **The pinned numbers are measured, not derived, and they all move together.** Change the
  width of any pane and every `min-width` in the plan pane is wrong until re-measured.

### Nothing offers what it cannot do

Both states are always mounted, so every control is always in the DOM whether or not it means
anything yet. An affordance that is wrong for the current state is withdrawn in the logic —
pointer *and* keyboard together — not left to look enabled. This is about affordances wrong for
the current state, not unimplemented chrome: the bell and the kebab may hover and do nothing.

### Two kinds of waiting, and they must never gate each other

A plan being negotiated (`Not agreed`, `ph-warning`) and a step owing a detail (`N to settle`,
`ph-question-mark`) are different acts. **Agreeing is never gated on answering.** Both are
counted in both views, in their own units.

### The wash means exactly one thing: this is waiting on your reply

Only the contested step's card and the one open question may carry the orange. *Which* reply it
is comes from the glyph and the word, never the colour; the colour only ever says *your move*.
Selection is blue, and never spends orange.

### One status word per state, across both views

Two views of one object that name it differently read as a change of subject — which is exactly
what the one-beat transition exists to avoid.

### Type: pick a role from the ladder, never invent a size

**There is no 11px and no 13px.** Spacing is the FlowX scale, Open Sans is plan language and
mono is everything the product says *about* the plan, and **there are no elevation shadows**.

## Preview

```
python3 -m http.server 8765 --bind 127.0.0.1
```

Always over HTTP — `file://` breaks `support.js` and the local fonts. Icons come from a CDN,
so they need network.

## Git

- Repo: <https://github.com/bogdandraghici/AI-Assisted-Builder> — **public**, branch `main`
- Pages: served from `main` at the repo root →
  <https://bogdandraghici.github.io/AI-Assisted-Builder/>, redeploying on every push
  (~20s). `.nojekyll` keeps files served verbatim.

**Never push unless I explicitly ask.** Commit freely — commits are local and
reversible — but a push publishes to a public URL that gets indexed. Prefer `git revert`
over rewriting history; no force-pushing what is already published.
