# The plan lies down

Folding the run into the prototype as three more of its states, settled 2026-07-30.

Companion to [2026-07-30-test-what-exists-design.md](2026-07-30-test-what-exists-design.md),
which designed the run as its own screen. This one makes it reachable: the run stops being
screen 02 and becomes the third state of the one prototype.

## What is wrong today

`Test what exists` is the green `ph-play` in the top bar at source line 80. It hovers and
does nothing. The run exists — fully designed, three states of it — but the only way to
reach it is `index.html`, which is the doc's job, not the product's. So the prototype
advertises an act it cannot perform, which is the one thing
[CLAUDE.md](../../../CLAUDE.md) says may not happen.

The return trip is broken in the same way from the other end. Screen 02's back arrow and
`The plan` crumb are live and honest about it in their own comment — *"it resets the run to
its edge rather than navigating, because leaving the doc is index.html's job"* — but what
they promise is the plan, and the plan is on another page.

## The decision: states, not screens

Blocks `01` and `02` merge into one `data-screen-label` block. `build-screens.py` then emits
one page and sweeps `screen-02.dc.html` on its next run, which is already its documented
behaviour for a screen removed from the source — only pages carrying the `GENERATED` banner
are swept, so nothing hand-written is at risk.

This is not a new principle, it is the existing one applied. CLAUDE.md: *"A screen with two
states is **one** `data-screen-label` block, not two … Two blocks would mean two copies of
the same markup, and they drift — that is how the chat panes ended up misaligned once
already."* The two chat panes are still two copies today, and the source itself flags the
history: *"The two chat panes drifting apart has already been a bug here twice, and the only
defence is that they are the same text at the same width in the same order."* Merging
retires that defence in favour of there being one pane.

The doc also already asserts the thing this change makes true. Source line 1298, of the run
strip: **"Same object as the plan, same words, lying down."** Today that is a claim about two
sets of elements. After this it is a description of one.

### State

One key joins, and the keys stay disjoint as they already do:

```js
run: get('run') === '1' || has('at') || has('full'),
```

`step`/`tech` remain the plan's, `at`/`full` remain the run's. `at` or `full` in the URL
implies `run`, so `index.html`'s five sections become:

| § | URL | State |
|---|---|---|
| 01 | `screen-01.dc.html` | the plan |
| 02 | `screen-01.dc.html?step=3` | a step open beside the spine |
| 03 | `screen-01.dc.html?run=1` | the run, at its edge |
| 04 | `screen-01.dc.html?run=1&at=1` | the run, scrubbed back |
| 05 | `screen-01.dc.html?run=1&full=1` | the run, filling the screen |

Their titles and descriptions do not change — the index owns those and they are still true.

## Why the region has to become a grid

Screen 01's shell is `[chat 360][plan pane][step pane]`, a flex row. Screen 02's is
`[chat][app column {frame / strip}]`. Merged, the plan has to travel from *a column where
the app's place is* to *a band beneath the app*. That is a change of axis, and
`flex-direction` is discrete: it would jump on frame one.

So everything right of the chat becomes **one grid, two rows by two columns, with fixed
placement and interpolated tracks** — the mechanism every collapse in this document already
uses:

- app frame at (row 1, columns 1–2)
- plan surface at (row 2, column 1)
- step pane at (row 2, column 2)

Nothing is ever re-placed. Only the tracks move, and `grid-template-rows` /
`grid-template-columns` interpolate cleanly between equal track counts — `1fr → 0fr` is
already load-bearing here in six places.

**Both tracks in a template must carry the same unit.** `780px → 1fr` does not interpolate
at all; it snaps. So every track is px, measured, which is this document's habit anyway.

| | rows: app / band | cols: plan / step |
|---|---|---|
| plan | `0 · H` | `1080 · 0` |
| step open | `0 · H` | `300 · 780` |
| run | `F · S` | `1080 · 0` |
| run + full | `F' · S` | `1440 · 0` |

`H` is the region's full height, `F`/`F'` the frame's two heights, `S` the band's. The known
anchors from the companion spec: the region is **1080** wide (`360 + 1080 = 1440`), inner
**1032** after the stage's 24px padding, and the frame is **1032 × 491** at rest going to
**1392 × 804** full. `H`, `S` and the exact `F` are outputs of the measurement pass below,
not numbers to carry over from prose — motion.md is explicit that these are *"a function of
the chat pane's width, so anything that moves that pane invalidates"* them.

## The five steps are five elements

Flow layout inside the plan surface **does not change**. The vertical column stays exactly
as it is, which is what lets the plan↔spine move keep every measured pin — the
`876 / 876 / 945 / 848 / 875 / 1015` set and the `min-height: 3lh` / `2lh` reservations — and
what makes its two resting states diff to zero pixels against this build.

The run is the displaced state. Each step keeps its flow position and carries
`transform: translate(xᵢ, −yᵢ)` plus an animating `width` to lie down. Transforms do not
affect layout, so nothing below them reflows; the band clips with `overflow: hidden` and the
surface's own height comes from the grid track, not from its content.

The group headings travel the same way, as one element each: `Agreed`, `Not agreed`, `Next`
are the same three words in the same three colours in both states, interleaved between rows
in the plan and ranged above the columns in the strip.

### The connector swings, it does not reshape

In the plan the connector drops from the dot; in the strip it runs right from it. Given the
same element and naive interpolation, `2 × 80` to `180 × 2` passes through roughly `91 × 41`
— a growing rectangle, which reads as a block appearing rather than a line turning. So it
stays a line and rotates about the dot: `rotate(-90deg → 0deg)` with its length animating.

The dot is absolutely positioned within its step. It carries no text, so moving it from the
left gutter to above the title costs no reflow and needs no pin.

### The title rewraps, and that is the one real hazard

A step title goes from ~780px of measure to ~203px. motion.md is unambiguous about what that
is: *"Rewrapping is not motion, it is a stack of 18px jumps"* — one jump per line gained, and
no curve smooths a jump.

It is fixed the way the collapse already fixes it, with both halves of the existing trick:

- **`max-width` leads the measure** on `cubic-bezier(0, 0, 0.4, 0.8)`, so it is always the
  smaller of the two constraints and it alone decides where lines break.
- **The line box is reserved before the text needs it**, `min-height: Nlh` animating ahead
  of the wrap opening and lagging the un-wrap closing. The strip's titles already hold
  `min-height: 3lh`, so the target end is known.

The gate on this is box-underrun, not eyeballing: `scrollHeight - min-height` over the five
titles, worst case, at every end. Sub-line means no title ever gains a line mid-flight.

## The chat

The delta between the two panes is small, and all three parts of it are already-solved
shapes:

1. The pane wrapper's full-screen machinery (`chatW`, `chatBorder`, `paneVis`, `paneOp`) is
   screen 02's and comes across as-is.
2. One turn appears — Agent · 09:46, *"I ran what exists…"*. It is always mounted and
   collapses `1fr → 0fr` outside the run, so it is not an `sc-if` and has a height to travel
   from. The pane has no scroller and the companion spec measured its budget at 723px against
   676px of content, so the turn fits at the run end — and the plan end is that same pane with
   one turn fewer, so it cannot be the tighter of the two. If the budget is ever exceeded,
   shorten the copy; do not add a scroller.
3. The composer caption changes, `lands on the step it is about` → `lands on the step you are
   looking at`. One element, interpolated text, and it sits in a block that is not moving.

## Four calls, and the reason for each

- **`Test what exists` goes quiet while a step is open**, gated with `pointer-events` and
  `tabindex` exactly as the back arrow and crumb are. Not tidiness: entering the run from the
  spine would superpose a spine→wide *flow* travel against the lie-down *transform*, and the
  sum of two curves is not guaranteed monotonic. A long late reversal is the one thing this
  move may not do — motion.md spent a retune of `ldDur` on 19px of exactly that. The run is
  entered from and returns to the wide plan, so there is one pair of endpoints to measure
  rather than two, and neither is superposed.
- **The identity line does not swap.** `build 2 · 09:37 · 7 resources exist` stays and a
  collapsing cell opens after it carrying the run's facts — the same animated-grid pattern the
  step crumb uses at the other end of the same bar. It is also the truer sentence: it is still
  build 2.
- **The green button keeps one label in both states.** Animating its width would move the top
  bar's right cluster, and the top bar is the one thing on this screen that does not move;
  swapping the word instantly inside a box whose width is animating pops. It reads `Test what
  exists` throughout, which is what it does in both states. **This is the one resting state
  that changes** — screen 02 loses the words `Run again` — and it is the deliberate cost of
  the bar holding still.
- **Esc from the run returns to the plan**, ordered after `full` and before `step` in the
  existing handler, so the most modal thing still closes first. With the arrow and the crumb
  that makes three ways back out of the run, matching the three out of a step.

## Two act rows, one set of gates

`Agree` / `Comment further` exists three times today: in step 3's plan card, in the step
pane's pinned footer, and in the strip's pinned act row. A single travelling element was
never going to cover all three — the step pane's lives in a different grid cell — and a pair
inside step 3's card would have to travel relative to a parent that is itself travelling,
which is the same superposed-transform risk ruled out above.

So each stays pinned in its own state, and they share **one** set of `pointer-events` /
`tabindex` gates and **one** caption string in `renderVals`. That is the defence the document
already uses for controls it cannot merge — *"they share `exitTab` / `upPE` and they arrive
and leave together"*, and *"they share one block and the button is gated, rather than the
block being written twice and drifting."* What may not differ between them is a word.

## What does not change

- Both existing resting states of screen 01, to the pixel.
- The plan↔spine move: same durations, same easings, same pins, same measured travel.
- The full-screen move: `1032 × 491 → 1392 × 804` on the shared 340ms curve.
- Every status word. A step is `Not agreed` in the plan chip, the spine label, the plan
  subtitle, the step badge and now the strip node; open questions are counted *to settle* in
  every view. One wash still means one thing, and the run still carries step 3's.
- The scrub (`at`) and its gating of nodes 4 and 5.

## Verification

Ordered, and each gate is a comparison against **this** build on **this** rig — motion.md is
explicit that the absolute figures in prose are rig-specific and that the only sound gate is
before/after on one machine.

1. **Measurement pass first.** A probe gated on all three of `.sc-placeholder` gone,
   `sc-dc-streaming` off and non-zero rects — *and* on the target element's own `style`
   attribute being free of `{{`, never on `document.body.innerHTML`, which contains the
   probe's own braces. It must fail loudly on impossible geometry: a check that prints PASS
   over all-zero rects is worse than no check. Gate on `document.fonts.ready` plus an
   explicit `fonts.load()` of the plan's weights, or the counts columns wobble run to run.
   Outputs: `H`, `S`, `F`, the five `(xᵢ, yᵢ)` pairs, the three heading pairs, and the
   titles' two measures.
2. **Resting states.** Screenshot the plan and the step-open states, diff against this build,
   expect **zero** differing pixels. Then the run's three states against `screen-02.dc.html`,
   expecting zero except the green button's label.
3. **Box-underrun at every end.** With transitions disabled, toggle to each of the four
   resting states and take `scrollHeight - offsetHeight` over every pinned box — the five
   titles, the boundary sentence, the chat's two blocks, the collapsing descriptions. Nothing
   smaller than its content at any end means nothing can gain a line between them.
4. **Stepped trace of plan↔run.** Click, give it a turn to render, then `getAnimations()`,
   `pause()`, and step every animation's `currentTime` together in 16.7ms increments, reading
   each row's `getBoundingClientRect().top`. Diff consecutive frames and **sum** the backward
   deltas per row as well as taking the worst single one — nineteen frames of 1px the wrong
   way is the same judder as one frame of 19px, and only the total catches it. A sign change
   is worse than a big step.
5. **Same trace for plan↔spine**, against this build, to prove the existing move did not
   move.

If a stepped trace is unavailable — headless Chrome settles transitions before
`getAnimations()` is asked, and `--virtual-time-budget` advances layout but not compositor
opacity — then check **both ends instead of the middle** per step 3, which establishes the
same property the trace exists to establish.

## Where this stands

Built and verified (commit `022d7b7`):

- One block, one chat, one top bar. `screen-02.dc.html` swept; `index.html`'s five
  sections are five states of one page.
- The 2×2 grid with the measured tracks above. All of them confirmed against the render:
  region 1080 × 852, rows `0/852 · 531/321 · 852/0`, columns `1080/0 · 300/780 · 1440/0`.
- The state machine: entry, `Fill the screen`, Esc unwinding one thing at a time, the
  breadcrumb, the back arrow, and `Test what exists` withdrawn over a step
  (`pointer-events: none`, `tabindex: -1`). All asserted by `probe-click.html`.
- Resting states against the previous build: **the plan is pixel-identical, zero
  differing pixels.** The step state differs only where the entry button goes quiet. The
  run's three states differ only in the top bar and in the corrected chat wrap.

**Not yet built: the five rows are still two sets of elements.** So the plan → run
transition is currently the collapse-and-fade this design rejected, not the plan lying
down. The mechanism below is settled and measured; it is the assembly that is outstanding.

### The remainder, as designed

Each step becomes `position: relative`, with:

- **`width`** — `100%` in the plan, `calc((100% - 16px) / 5)` in the run. Both are
  interpolable, and neither is a measured constant, so the band still divides correctly
  above 1440 the way `flex: 1 1 0px` did.
- **`transform`** — `translateX` in multiples of the element's own width, which is
  viewport-independent: `0`, `100%`, `200%`, `calc(300% + 16px)`, `calc(400% + 16px)`,
  the 16px being the wall's gap. `translateY` is the negative of the cumulative flow
  height above each step *in the run state*, which is content-dependent and must be
  measured after the internals collapse — the one set of numbers still to take.
- **The dot** — absolutely positioned, so moving it from the left gutter to above the
  title costs no reflow. `top: 0` in both, because absolute positioning resolves against
  the padding box and so is unaffected by the step's own `padding-top`. The plan's step-4
  and step-5 dots are already `1px dashed #8b98a5` and `1px solid #8b98a5`, exactly the
  strip's, so only `dotSize`, `left` and `top` interpolate.
- **The connector rotating rather than reshaping**, and without measured lengths. All
  four insets are set in both states so neither is ever `auto`, and the two that carry
  the length use percentages: plan `left 9 · top 24 · right calc(100% - 11px) · bottom 0`,
  run `left 20 · top 9 · right 0 · bottom calc(100% - 11px)`. Those interpolate as calc,
  so the line auto-sizes to its row in the plan and to its node in the run with no number
  to keep in sync.
- **`padding: 28px 8px 0 0`** in the run — the 20px dot plus the card's 8px top margin,
  and the card's 8px right margin.
- **The title** — `max-width` 800 → 224/206 → 169, leading the measure on
  `cubic-bezier(0, 0, 0.4, 0.8)`, with `min-height` reserving the line box as it already
  does at `3lh`.

The three group headings travel the same way; the strip's own heads row and nodes row are
then deleted, since those elements come from the plan, and its wall, boundary caption and
act row move into a run-only wrapper.

## Risk, stated plainly

This is the largest single change to `Conversational Builder v2.dc.html`: two 1440×900 blocks
become one, the right-hand shell changes from flex to grid, and a new move joins the two that
are already tuned. The mitigation that matters is that the *existing* move's basis is
untouched — flow layout stays the plan's column, and the run is the transformed state — so
step 5 of the verification is a real gate rather than a formality. If it fails, the failure
is in the new work, not in a rebuilt foundation.
