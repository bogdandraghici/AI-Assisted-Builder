# Nothing offers what it cannot do

Both states are always mounted, so every control is always in the DOM whether or not it
means anything yet. An affordance that is wrong for the current state has to be withdrawn
in the logic, not left to look enabled.

This is about affordances that are **wrong for the current state**, not about unimplemented
chrome. The bell, the kebab and the undo pair hover and do nothing, and that is the fidelity
of a design doc — leave them. What may not happen is a control that looks live in a state
where it cannot act.

## How to withdraw one

- **Gate with `pointer-events`, in the interpolated `style` attribute.** `style-hover` /
  `style-active` / `style-focus` are passed to `pseudoClass()` verbatim (`support.js:428`)
  and take **no** `{{ }}` placeholders, so a hover can only be withdrawn by taking the
  pointer off the element — which removes the cursor and the click with it. One flag, whole
  affordance.
- **Withdraw the tabindex too**, or the keyboard keeps reaching a control the mouse no
  longer can — and the rule runs both ways: never leave a pointer target the keyboard cannot
  reach either, which is how the previewed app's live control was mouse-only for a while. A
  whole clipped pane does both at once: it goes out of the tab order with `visibility` on the
  pane — it inherits, so it does not need a tabindex on every control inside, and being
  discrete it cannot judder. Delay the hide by the content's fade-out so it does not pop.

## `{{ … }}` resolves in *every* attribute

`collectProps` runs `compileAttr` over all of them (`support.js:441`), so `tabindex`, `title`
and `class` interpolate exactly like `style` does. The `style-hover` / `style-active` /
`style-focus` trio above is the one exception, and it is an exception because those are
passed to `pseudoClass()` verbatim rather than compiled.

So a state-dependent tab stop, tooltip or glyph is **one element with an interpolated
attribute**, never two behind an `sc-if`.

*Do not re-diagnose this from a rendered DOM without gating on hydration properly* — a probe
that checks only for `.sc-placeholder` reads the raw markup, sees `tabindex="{{ exitTab }}"`,
and will convince you of a bug that is not there. It did once. See
**[motion.md](motion.md)**, "How to measure it".

## Three glyphs, three meanings

- `caret-right` is a disclosure that opens in the pane you are already in.
- `arrow-right` changes what you are looking at.
- `arrows-out` / `arrows-in` changes how *much* of it you are looking at.

A control and its row carry the same one, and the expand pair holds the same slot in both
states so the way back is never somewhere the way in was not.

The question-mark and warning glyphs carry the two kinds of waiting — see
[waiting.md](waiting.md).

## The four ways out of a step are one control at four sizes

The three ways out — ×, Esc, breadcrumb — plus the back arrow share `exitTab` / `upPE`, and
they arrive and leave together. Up from the plan is a journey list this prototype does not
have, so in the plan state all of them go quiet rather than promise it.

## One status word per state, across both views

A step is `Not agreed` on the wide plan's card heading, on its own eyebrow in the spine and in
the step badge; open decisions are counted as *to settle* in every view. Two views of one
object that name it differently read as a change of subject, which is exactly what the
one-beat transition is built to avoid. The plan's *grouping* is a different axis and says a
different word — `Built` / `Not built yet`, see [waiting.md](waiting.md) — which is what lets
one step be built and not agreed and still owing an answer without any of the three lying.
