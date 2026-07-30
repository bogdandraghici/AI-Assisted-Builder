# Two kinds of waiting, and they must never gate each other

The builder is asked for two different things and they are not the same act:

- **The plan is being negotiated.** The agent will not do what was asked and says why.
  Word: `Not agreed` → `Agreed`. Glyph: `ph-warning`. Argued in the chat, settled in the
  step's footer.
- **A step needs a detail before it can be built.** Word: `N to settle` / `Not answered`.
  Glyph: `ph-question-mark` — *not* `ph-question`, which is the top bar's help button.
  Answered in place, on the question itself.

## The wash means exactly one thing: this is waiting on your reply

Both kinds wear the orange wash, and only the two objects above may carry it — the contested
step's card, in the plan and in the spine, and the one open question in the step pane. So two
washes on screen with a step open is right, and says you owe two replies. Which reply is the
glyph and the word, never the colour; the colour only ever says *your move*.

Nothing else takes the wash — the 180° gradient behind a tinted edge is those two objects and
no others. Colour alone is a narrower claim than the wash, and exactly one count spends it:
the plan row's settle tag, below. There are no left status stripes — see
[type-and-palette.md](type-and-palette.md) for what that key cost, why the wash replaced it,
and the wash's own values.

## Rules that follow, each of them once broken here

- **Agreeing is never gated on answering.** They are independent, and the plan says so
  itself: steps 4 and 5 are under *Next*, agreed, each still owing answers.
  Agreed-and-still-owing is a legal state, so a contested step must be able to reach it.
  `Agree and build` welded the two together and then went dead, which made the one thing the
  screen exists to ask unanswerable until unrelated spec work was done. The two acts of an
  argument are **agree** and **comment further**; *build* is the agent's move, not the
  builder's, and it belongs in no button here.
- **Both kinds are counted in both views, in their own units** — the argument in steps, the
  questions in questions. Any partition of the five steps into one bucket each cannot say
  that a contested step also owes an answer, so the questions end up counted nowhere.
- **A run of what exists may carry the wash for the step it is evidence about.** Screen 02
  washes step 3 in its rail and carries `Agree` / `Comment further` there, because the run
  has just demonstrated the claim under dispute: the check went out unseen in 3s, which is
  the whole of *"it slows the customer down"*, answered. It was built at 09:37, two minutes
  before you asked to drop it at 09:39 — that timeline is what earns the screen its subject.
  Wash it where the reply is given, and the reply is best given here. Still one wash on that
  screen, still the same object the plan washes, still the same words in all three views.
- **One tag per plan row, and it is the whole of the row's right-hand column.** `N to settle`
  or `Nothing to settle`, and nothing else. It replaced `N resources ›`, which counted the
  agent's own output in a unit the reader has no way to judge — three resources is neither
  good nor bad — and spent the one column a row has on a disclosure rather than on the row's
  state. The plan is read for what is still owed, so that is what the column says. The
  header's `Every version` went with it: the plan's history is a different pane's job, and
  the step pane already keeps a version list with a scope worth having.
- **The settle tag is the one mention that wears orange, and it wears a flat tint, never the
  wash.** Orange when something is owed, grey when nothing is; the same chip either way, so
  the count and the `ph-question-mark` still say *which* kind of waiting it is and the colour
  says only whether there is any. This overturns the earlier rule that no mention of a
  question could be orange — the argument being that orange means *your move* and a count is
  a fact about the plan. What changed is what the column is: it is now the list of what is
  owed, read down in one pass, and a row owing nothing has to be distinguishable from a row
  owing two without reading five numbers. The wash's own meaning is untouched: still a
  gradient, still two objects, still the only thing that says *answer this here*.
- **The contested step's chips stay grey, and it is the one exception.** Row 3 already spends
  orange twice — the wash and the `Not agreed` heading — and a third orange thing meaning
  something else on one card is the two kinds of waiting wearing one colour. It says its open
  question in words, in the card's footer caption, instead.
- **A mention still gets the question's glyph, in its own colour.** Every "N to settle" count
  and caption carries a `ph-question-mark` taking the colour of whatever it precedes — the
  tag's orange, the count's grey everywhere else. The glyph never has a colour of its own.
- **Selection never spends orange.** The step open beside the spine wears a 1px `#4d97ea`
  outline — blue is "you are here", orange is "your move", and they may stack. See
  [type-and-palette.md](type-and-palette.md).
- **The act is pinned, never scrolled.** The step pane's `Agree` / `Comment further` row sat
  38px below the fold at 1440×900 with 143px of overflow above it, so the panel asked you to
  decide and put the decision off-screen — which reads as the call not being there at all. It
  is now outside the scroller, for the same reason the × is. Both panes are then one shape: a
  scrolling body with the act pinned beneath it. If that row grows, the scroller's `bottom`
  is the number to change with it.

## Full screen is the chat's one exception

The chat is a fixed 360 in every other state on every screen, because a pane that resizes for
no reason is one more thing travelling during a move. Asking to see only the app is a reason:
the chat and the run strip collapse together on the shared 340ms curve, and everything the
collapse passes over is pinned so it clips instead of reflowing — the chat's two blocks at
`min-width: 360px`, the strip's five titles at `min-height: 3lh`, its boundary sentence at
`2lh`. The act goes with the strip, which is honest rather than lossy: you asked for only the
app, and Esc brings it back.
