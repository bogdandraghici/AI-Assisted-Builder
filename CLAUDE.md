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

Build any new page with both variants in mind. Design size is 1440×900; standalone pages
fill the viewport but hold a `min-width` / `min-height` at that size — below it the panes
and table columns collide, so the page scrolls instead of breaking.

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

The two states are joined by a transition, and that transition is the most delicate thing
in the repo. **[docs/motion.md](docs/motion.md)** holds it: why it is one beat, the five
things that make one beat possible, the measured `min-width` / `min-height` numbers, the
easing, and how to prove a change did not make it judder. Read it before touching any
duration, easing, collapse or pinned measurement in the plan pane.

Three of its rules are easy to break without opening it:

- **Never `sc-if` anything that animates** — an unmounted element has no value to
  transition from. `sc-if` is for things that just appear.
- **One element per thing, never two that cross-fade.** The wide plan and the 300px spine
  are the same five rows changing size, not two layers dissolving.
- **The pinned numbers are measured, not derived, and they all move together.** Change the
  width of any pane and every `min-width` in the plan pane is wrong until re-measured.

## Nothing offers what it cannot do

Both states are always mounted, so every control is always in the DOM whether or not it
means anything yet. An affordance that is wrong for the current state has to be withdrawn
in the logic, not left to look enabled:

- **Gate with `pointer-events`, in the interpolated `style` attribute.** `style-hover` /
  `style-active` / `style-focus` are passed to `pseudoClass()` verbatim
  (`support.js:428`) and take **no** `{{ }}` placeholders, so a hover can only be
  withdrawn by taking the pointer off the element — which removes the cursor and the click
  with it. One flag, whole affordance.
- **Withdraw the tabindex too** (`tabindex="{{ … }}"` → `-1`), or the keyboard keeps
  reaching a control the mouse no longer can. A whole clipped pane goes out of the tab
  order with `visibility` on the pane — it inherits, so it does not need a tabindex on
  every control inside, and being discrete it cannot judder. Delay the hide by the
  content's fade-out so it does not pop.
- **Two glyphs, two meanings.** `caret-right` is a disclosure that opens in the pane you
  are already in; `arrow-right` changes what you are looking at. A control and its row
  carry the same one.
- This is about affordances that are **wrong for the current state**, not about
  unimplemented chrome. The bell, the kebab and the undo pair hover and do nothing, and
  that is the fidelity of a design doc — leave them. What may not happen is a control that
  looks live in a state where it cannot act.
- **The three ways out of a step — ×, Esc, breadcrumb — plus the back arrow are one
  control at four sizes.** They share `exitTab` / `upPE` and they arrive and leave
  together. Up from the plan is a journey list this prototype does not have, so in the
  plan state all of them go quiet rather than promise it.
- **One status word per state, across both views.** A step is `Not agreed` in the plan
  chip, in the spine label, in the plan subtitle and in the step badge; open decisions are
  counted as *to settle* in both panes. Two views of one object that name it differently
  read as a change of subject, which is exactly what the one-beat transition is built to
  avoid.

## Type and spacing

**[docs/type-and-palette.md](docs/type-and-palette.md)** — the type ladder, the spacing
scale, the two families and the palette, all from `Design System.dc.html`. In short: pick a
role from the ladder and never invent a size (**there is no 11px and no 13px**), spacing is
the FlowX scale, Open Sans is plan language and mono is everything the product says *about*
the plan, and **there are no elevation shadows**.

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
