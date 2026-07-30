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
- **`{{ … }}` resolves in *every* attribute** — `collectProps` runs `compileAttr` over all
  of them (`support.js:441`), so `tabindex`, `title` and `class` interpolate exactly like
  `style` does. The `style-hover` / `style-active` / `style-focus` trio above is the one
  exception, and it is an exception because those are passed to `pseudoClass()` verbatim
  rather than compiled. So a state-dependent tab stop, tooltip or glyph is **one element
  with an interpolated attribute**, never two behind an `sc-if`.
  *Do not re-diagnose this from a rendered DOM without gating on hydration properly* — a
  probe that checks only for `.sc-placeholder` reads the raw markup, sees
  `tabindex="{{ exitTab }}"`, and will convince you of a bug that is not there. It did once.
  See **[docs/motion.md](docs/motion.md)**, "How to measure it".
- **Withdraw the tabindex too**, or the keyboard keeps reaching a control the mouse no
  longer can — and the rule runs both ways: never leave a pointer target the keyboard
  cannot reach either, which is how the previewed app's live control was mouse-only for a
  while. A whole clipped pane does both at once: it goes out of the tab order with
  `visibility` on the pane — it inherits, so it does not need a tabindex on every control
  inside, and being discrete it cannot judder. Delay the hide by the content's fade-out so
  it does not pop.
- **Three glyphs, three meanings.** `caret-right` is a disclosure that opens in the pane
  you are already in; `arrow-right` changes what you are looking at; `arrows-out` /
  `arrows-in` changes how *much* of it you are looking at. A control and its row carry the
  same one, and the expand pair holds the same slot in both states so the way back is never
  somewhere the way in was not.
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

## Two kinds of waiting, and they must never gate each other

The builder is asked for two different things and they are not the same act:

- **The plan is being negotiated.** The agent will not do what was asked and says why.
  Word: `Not agreed` → `Agreed`. Glyph: `ph-warning`. Argued in the chat, settled in the
  step's footer.
- **A step needs a detail before it can be built.** Word: `N to settle` / `Not answered`.
  Glyph: `ph-question-mark` — *not* `ph-question`, which is the top bar's help button.
  Answered in place, on the question itself.

Both wear the orange wash, and **the wash means exactly one thing: this is waiting on your
reply.** Only the two objects above may carry it — the contested step's card, in the plan and
in the spine, and the one open question in the step pane. So two washes on screen with a step
open is right, and says you owe two replies. Which reply is the glyph and the word, never the
colour; the colour only ever says *your move*. Everything else stays quiet: a count of open
questions in a plan row is grey mono, because a count is not a question you can answer where
it stands. There are no left status stripes — see
[docs/type-and-palette.md](docs/type-and-palette.md) for what that key cost and why the wash
replaced it.

Three rules follow, and each of them was once broken here:

- **Agreeing is never gated on answering.** They are independent, and the plan says so
  itself: steps 4 and 5 are under *Next*, agreed, each still owing answers.
  Agreed-and-still-owing is a legal state, so a contested step must be able to reach it.
  `Agree and build` welded the two together and then went dead, which made the one thing
  the screen exists to ask unanswerable until unrelated spec work was done. The two acts of
  an argument are **agree** and **comment further**; *build* is the agent's move, not the
  builder's, and it belongs in no button here.
- **Both kinds are counted in both views, in their own units** — the argument in steps, the
  questions in questions. Any partition of the five steps into one bucket each cannot say
  that a contested step also owes an answer, so the questions end up counted nowhere.
- **Full screen is the chat's one exception.** The chat is a fixed 360 in every other
  state on every screen, because a pane that resizes for no reason is one more thing
  travelling during a move. Asking to see only the app is a reason: the chat and the run
  strip collapse together on the shared 340ms curve, and everything the collapse passes
  over is pinned so it clips instead of reflowing — the chat's two blocks at
  `min-width: 360px`, the strip's five titles at `min-height: 3lh`, its boundary sentence
  at `2lh`. The act goes with the strip, which is honest rather than lossy: you asked for
  only the app, and Esc brings it back.
- **A run of what exists may carry the wash for the step it is evidence about.** Screen 02
  washes step 3 in its rail and carries `Agree` / `Comment further` there, because the run
  has just demonstrated the claim under dispute: the check went out unseen in 3s, which is
  the whole of *"it slows the customer down"*, answered. It was built at 09:37, two minutes
  before you asked to drop it at 09:39 — that timeline is what earns the screen its subject.
  Wash it where the reply is given, and the reply is best given here. Still one wash on that
  screen, still the same object the plan washes, still the same words in all three views.
- **Never spend orange on a mention of a question — only on the question itself.** The four
  routine questions are grey counts in their plan rows; the one you can answer where it
  stands is washed. Orange every mention of them and it stops meaning *your move* and starts
  meaning *questions exist*, which is a fact about the plan, not a call on you.
- **A mention still gets the question's glyph — in grey.** Every "N to settle" count and
  caption carries a `ph-question-mark` in the count's own grey, so the question kind of
  waiting is recognisable on the plan without spending orange. The orange glyph appears
  only on the answerable card in the step pane.
- **Selection never spends orange.** The step open beside the spine wears a 1px `#4d97ea`
  outline — blue is "you are here", orange is "your move", and they may stack. See
  [docs/type-and-palette.md](docs/type-and-palette.md).
- **The act is pinned, never scrolled.** The step pane's `Agree` / `Comment further` row sat
  38px below the fold at 1440×900 with 143px of overflow above it, so the panel asked you to
  decide and put the decision off-screen — which reads as the call not being there at all.
  It is now outside the scroller, for the same reason the × is. Both panes are then one
  shape: a scrolling body with the act pinned beneath it. If that row grows, the scroller's
  `bottom` is the number to change with it.

Adding a count to a plan row is not free — see [docs/motion.md](docs/motion.md). It widens
the counts column, which narrows the title beside it at every instant of the move, and a
title whose real width drops below its own animating `max-width` hands the wrapping back to
the pane. **Stack the count under the build phrase rather than beside it**, so the column
stays as wide as its widest line and every `min-width` pin holds.

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
