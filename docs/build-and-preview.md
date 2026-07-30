# Building and previewing the pages

## The two variants

Every page ships twice:

1. **Standalone** — the page on its own, filling the viewport. No caption, heading,
   description or card framing. What it would look like in the real product.
2. **Embedded** — framed inside `index.html` at its design size and scaled to the column.
   The index owns the title and description, so the standalone page never repeats them.

Design size is 1440×900. Standalone pages fill the viewport but hold a `min-width` /
`min-height` at that size — below it the panes and table columns collide, so the page scrolls
instead of breaking.

## Generated screens

Standalone screens are generated, not hand-written:

```
python3 build-screens.py
```

It splits each top-level `data-screen-label` block out of
`Conversational Builder v2.dc.html` into `screen-NN.dc.html`, stripping the doc framing
(caption, black canvas, padding, 1440×900 card with border and radius).

- Edit the source doc, never `screen-*.dc.html` — they are overwritten on every run and carry
  a `GENERATED` banner.
- The script hard-fails with a clear message if the source is restyled such that it can no
  longer strip the framing, rather than emitting a broken page.
- Add a screen to the source → re-run, then add its `index.html` section by hand.
- Remove a screen from the source → re-run and its page is deleted. Only pages carrying the
  banner are swept, so a hand-written `screen-*.dc.html` is safe.

## How one page shows two states

A screen with two states is **one** `data-screen-label` block, not two. The plan and the
plan-with-a-step-open are the same page; `index.html` shows both by embedding it twice, the
second time as `screen-01.dc.html?step=3`, which the `Component` reads off `location.search`.

Two blocks would mean two copies of the same markup, and they drift — that is how the chat
panes ended up misaligned once already.

The two states are joined by a transition, and that transition is the most delicate thing in
the repo: see [motion.md](motion.md).

## Preview

```
python3 -m http.server 8765 --bind 127.0.0.1
```

Always over HTTP. Opening `file://` breaks `support.js` and the local fonts. Open Sans ships
from `fonts/`; Phosphor icons come from a CDN, so icons need network.
