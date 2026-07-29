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

Two families, and the distinction is load-bearing: Open Sans is plan language, mono is
resource identifiers. Uppercase 10px is a panel or column header and nothing else.

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
