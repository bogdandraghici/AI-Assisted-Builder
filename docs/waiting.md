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

Everything else stays quiet: a count of open questions in a plan row is grey mono, because a
count is not a question you can answer where it stands. There are no left status stripes —
see [type-and-palette.md](type-and-palette.md) for what that key cost, why the wash replaced
it, and the wash's own values.

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
- **Never spend orange on a mention of a question — only on the question itself.** The four
  routine questions are grey counts in their plan rows; the one you can answer where it
  stands is washed. Orange every mention of them and it stops meaning *your move* and starts
  meaning *questions exist*, which is a fact about the plan, not a call on you.
- **A mention still gets the question's glyph — in grey.** Every "N to settle" count and
  caption carries a `ph-question-mark` in the count's own grey, so the question kind of
  waiting is recognisable on the plan without spending orange. The orange glyph appears only
  on the answerable card in the step pane.
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
