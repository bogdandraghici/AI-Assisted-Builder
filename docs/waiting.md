# Two kinds of waiting, and they must never gate each other

The builder is asked for two different things and they are not the same act:

- **The plan is being negotiated.** The agent will not do what was asked and says why.
  Word: `Not agreed` → `Agreed`. Glyph: `ph-warning`. Argued in the chat, settled in the
  step's footer.
- **A step needs a detail before it can be built.** Word: `N to settle` / `Not answered`.
  Glyph: `ph-question-mark` — *not* `ph-question`, which is the top bar's help button.
  Answered in place, on the question itself.

## The wash means exactly one thing: this is waiting on your reply

Both kinds wear the orange wash, and only the two objects above may carry it — the argument
about the contested step, and the one open question in the step pane. So two washes on screen
with a step open is right, and says you owe two replies. Which reply is the glyph and the
word, never the colour; the colour only ever says *your move*.

**The wash is on the argument, not on the step.** In the wide plan the contested step is an
ordinary row — title, description, settle tag, same left edge as the other four — and the
washed card sits *under* it holding the whole of what is owed: the objection, the reference,
the drawing, and the two acts. The card engulfing the step said the step was the thing
waiting on you, and it is not; the step is a step, and there happens to be an argument about
it. In the spine the card is inside the same collapse as every other row's description and
clips to nothing, so the wash falls back onto the step's own block and the spine keeps the
one washed card it has always had — one element in both states, only its paint crossing.

**The spine's wash carries its own `Not agreed`, and it has to.** It used to take that word
from the group heading above it, and the grouping is on the build axis now, so the heading
above it says `Not built yet` — a true thing about the step, and not this one. A wash with no
word beside it leaves the colour saying which kind of waiting this is, which is the one thing
colour is never allowed to do. So step 3's
spine block opens with a `Not agreed` eyebrow of its own on `rmRows` / `rmOp`, the same word
the wide plan's card heading and the step pane's badge carry. It is wrapped with the title in
a `gap: 0` box rather than dropped into the column beside it: a 0fr grid still pays its
column's 2px gap, which in the wide plan would push that one title 2px below its own dot
while the other four sat level with theirs. The 8px pad reads as exactly 8px, because the
title row's -6px margin and its own 6px padding-top cancel.

Nothing else takes the wash — the 180° gradient behind a tinted edge is those two objects and
no others. Colour alone is a narrower claim than the wash, and exactly one count spends it:
the plan row's settle tag, below. There are no left status stripes — see
[type-and-palette.md](type-and-palette.md) for what that key cost, why the wash replaced it,
and the wash's own values.

## The plan groups on the build, not on the agreement

**`Built` and `Not built yet`, and there is no third heading.** The spine and the run strip
both used `Agreed` / `Not agreed` / `Next`, which grouped the five steps by whether the plan
itself was settled and gave the contested step a bucket of its own. That spent a heading on a
state the step already says three other ways — an orange dot, a washed card, two acts — while
saying nothing about the question the plan is actually read for. `Next` was no better: it
named where three rows sat in a queue, which is the one thing the order already shows.

**`Built` is the strict word: finished, agreed, standing.** Two rows are under it. The
contested step is not one of them, and its own act has said so all along, in three places:
*one question still to settle **before I build it***. Its last two resources —
`enquiryToContext.dataMapper`, `enquiryOutcome.enum` — are `no` in the resource table, and the
shape being argued about is a revision that does not exist yet. So `Not built yet` opens with
step 3.

**The run strip's boundary is one column further right, and that is not a disagreement.** The
wall marks what *exists*, which is the looser claim and the only one a run can make: three
steps ran, two of them are built. Which is why the strip's left heading is `Ran` and not
`Built` — it is the one place the two views use different words, and the cards underneath
force it, because all three of them say `ran · …` and one of them is not built.
`Not built yet` is the plan's word unchanged, over the two columns it is true of. Merging
three headings into `flex: 3` + `flex: 2` left every column boundary where it was, spacer
included, so the wall at 0.6 still lands on the seam it names. `Run what is built` was the
run button's tooltip and is now `Run what exists`, for the same reason.

**The header's clauses do not partition the five steps, and are not meant to.** `Two of five
built · one not agreed · four to settle` is the build, the argument and the questions — three
axes, and step 3 sits on all three at once: the one row under `Not built yet` that is also the
one not agreed and also owes an answer. No bucket can say that; three counts in three units
can. The spine's `5 steps · 2 built` repeats the first clause instead of introducing
`2 agreed`, which landed on the same number for a different reason — the kind of agreement
that hides a bug rather than showing there is none.

**Orange leaves the group headings entirely,** which is stricter rationing than before rather
than looser: it is now the settle tags, step 3's dot, the wash, and step 3's own eyebrow.

The per-card `not built` caption on run columns 4 and 5 went with the change: under a
`Not built yet` heading it was the word said twice on one card, and each card carries one
caption either way — `ran · …` on the three that exist, what is owed on the two that do not.

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
- **One tag per plan row, and it is the whole of the row's right-hand column — all five of
  them.** `N to settle` or `Nothing to settle`, and nothing else. The contested row used to
  hold `Not agreed` there and no count at all, on the grounds that a count cost 74px of title
  measure; that was true while its title lived inside a padded card with the chip beside it,
  and it stopped being true when the row became an ordinary row paying exactly what rows 4
  and 5 pay. `Not agreed` is the heading of the card below, which is the object the state
  belongs to. The tag replaced `N resources ›`, which counted the
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
- **The contested step's chips stay grey in the spine, and it is the one exception.** There
  the step's own block is washed, under its own `Not agreed`, so it already spends orange
  twice and a third orange thing meaning something else on one card is the two kinds of
  waiting wearing one colour. In the wide plan the row carries no wash of its own, so its
  settle tag is orange like every other row's — the exception is about the card, not about
  the step, and it lives wherever the card is.
- **One chip in that row, not two.** A `2 not built` sat beside `1 to settle` — this step's
  own resource count, two of its five — and under a green `Built` heading it reads as a flat
  contradiction even though both are true. It counted the agent's output in a unit only the
  resource table can make sense of, and that table lists both rows with `no` against them.
  Dropping it also leaves the spine saying exactly what the wide row's tag says, so the two
  views agree chip for chip; and with one chip nothing wraps, so the 194px pin that kept the
  pair from breaking onto a second line mid-flight went with it.
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
