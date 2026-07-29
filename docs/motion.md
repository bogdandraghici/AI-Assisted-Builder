# The one-beat move

How the plan pane collapses into its 300px spine while the step pane opens beside it.
Read this before changing any duration, easing, `min-width`, `min-height` or collapse in
`Conversational Builder v2.dc.html` — every number in it was measured, and several of
them are only correct relative to each other.

## The mechanism

Motion is CSS transitions over state-driven inline styles — `style="flex-basis: {{ planBasis }}"`,
with `renderVals()` returning the value for each state. `support.js` re-resolves style
strings on every render, so this animates. Four rules follow from it:

- **Never `sc-if` anything that animates.** It unmounts, and an unmounted element has
  no value to transition from. `sc-if` is for things that just appear, like the
  technical-detail table.
- **One element per thing, never two that cross-fade.** The wide plan and the 300px
  spine are the *same* five rows changing type size, dot size and padding. Two layers
  dissolving into each other was the first attempt and it read as broken: you saw the
  wide plan clipped mid-sentence while a differently-shaped rail ghosted in under it.
  If the two states share a thing, it must be one element.
- **Collapse with `grid-template-rows: 1fr → 0fr`** (and `-columns` horizontally) on a
  wrapper whose child is `overflow: hidden; min-height: 0`. The closed size is then
  exactly the content's own height, with no magic number to keep in sync as copy
  changes — which `max-height` needs.
- **Fade before you close, fade after you open.** A column closing on live text leaves
  a one-character sliver, and that single frame is what makes a move look cheap. The
  `exOp*` values run shorter than the `ex*` ones for exactly this.

## Why it has to be one beat

The move is **one** beat. The panes resize while the detail collapses while the spine's
own labels open their space while the padding tightens — one curve, one duration, both
directions, everything starting at the same instant. It was three beats first, then two,
and both read exactly as the complaint each time: "it happens in two steps", then "there
is too much going on". A sequence is not something to schedule well; it is something to
remove, and two specific things force one if you let them.

### Reflowing content cannot collapse while the pane narrows over it — so stop it reflowing

A `1fr → 0fr` box's height is a fraction of its *content's* height, so a description that
rewraps mid-collapse makes that fraction grow while it is meant to be shrinking, and every
row below it lurches. That is the whole reason the collapse used to have to finish before
the panes could start. Give each collapsing block a `min-width` equal to the width it has
when open and it clips instead of reflowing — invisibly, since it is at zero opacity long
before the pane reaches it.

Measure the number, don't guess it: they differ per row
(**876 / 876 / 945 / 848 / 875 / 1015** at 1440) because the counts column beside each one
does. They are a function of the chat pane's width, so anything that moves that pane
invalidates all six — they all shifted 80px when the chat settled on one 360 width.

Measure with the webfonts *loaded*: gate the probe on `document.fonts.ready` plus an
explicit `fonts.load()` of the weights the plan uses, or a late-arriving face shifts every
counts column a few px and the numbers wobble run to run. The tell is that rows with no
counts column beside them stay stable while the others move.

### Rewrapping is not motion, it is a stack of 18px jumps

One jump per line a title gains, and no curve smooths a jump. The old answer was to hide
them inside a collapse violent enough to cover them, which is *why* the collapse was
violent: 60px in a frame. The real answer is to reserve the line box before the text needs
it — `min-height: 3lh` / `2lh` animating from `1lh`, leading the measure, so the wrap
happens inside a box that is already big enough. Five titles restacking then move nothing
at all. That one change took the interior rows from 15–25px lurches to a 4px-a-frame drift
and let the collapse slow down to 20px a frame.

Closing, it must *lag* the measure instead — the box may only shrink once the text has
un-wrapped, or the content sets the height again and the jumps come back. The `lh` unit
keeps it honest: if copy ever makes a count wrong, that title degrades to a jump, not to a
broken layout.

It has to lag the **un-wrap**, though, not the whole duration — and that distinction is the
difference between a move that closes well and one that does not. Closing once took the full
340ms on an ease that finished late, which put the panes at rest around 300ms while 32px of
reserved box per title was still collapsing. Rows that had travelled 218px down then drifted
**19px back up** over the last 100ms: a long, late reversal, which is the one thing this move
is not allowed to do. It read the way it measured — smooth opening, awkward closing.

Closing at 240ms fixes it by putting the shrink back inside the window where the pane is
still moving, so it is absorbed rather than exposed. The backward drift goes 19.4px → 1.4px
and the peak forward step 35px → 28px, which is the opening's own 27. Nothing about the rule
is weakened: the text un-wraps inside the first ~80ms, because `max-width` opens on the fast
part of its curve, so the box is never asked to be smaller than its content.

Check that with the **box-underrun** trace, not by comparing the two durations: at every
frame take `scrollHeight - min-height` over the five titles and keep the worst. It stays
around 2px — sub-line, so no title ever gains a line mid-flight. If a retune ever pushes
that past a line (18px), the jumps are back and the duration is too short.

### Let the measure outrun the pane and the pane never touches the wrapping

Both start together, but a title's `max-width` travels 800 → 224 while the space available
to it travels 879 → 224, so on the same curve the max-width is *always* the smaller of the
two and it alone decides where lines break. That is what makes a simultaneous move legal
at all.

It must equal the true final width, gaps included — a collapsed sibling still occupies its
flex `gap`, so the first attempt was 24px too generous and bought one last rewrap on the
final frame. Derive it, from the spine width in and the gaps out:
row `300 - 32 pad - 8 dot - 12 gap - 24 gap = 224`, and the card from the 248 the row
leaves it: `248 - 2 border - 20 pad - 12 gap - 8 gap = 206`. It applies to anything that
wraps, not just titles: the pair of chips in the card broke onto a second line 20ms before
the dock landed and threw the two rows below it down 26px in one frame. At 206 they fit
side by side and no longer wrap at all — but the pin stays, because what it prevents is the
wrap happening *during* the move, and an unpinned pair would still cross their own break
point on the way in.

### Whatever has no counterpart in the other state still has a SIZE, and the size is not late

The Agreed / You are here / Next headings and the per-step counts can only fade, but the
76px they occupy belongs to the shape the plan is collapsing into, so that space opens with
everything else and only the opacity waits. Given a beat of its own it made five rows fly
up past where they belonged and slide a hundred pixels back down once the rest had settled.

### Arrivals do not need a stagger to feel considered

The step's five blocks each slid up 10px in their own 40ms wave, which is five more things
moving while the plan is collapsing and the panes are resizing. One fade, no translate: the
pane opening is the motion, and the content is what it uncovers.

## Easing and durations

The FlowX `cubic-bezier(0.2, 0, 0.2, 1)` at 340ms for the move and 160ms for hovers. The
two things that lead or lag it — the measure and the reserved line box — use
`cubic-bezier(0, 0, 0.4, 0.8)` to get ahead early.

The chat pane is a fixed 360px in both states and takes no part in the move. The step pane
clips rather than reflows: its content holds a `min-width: 780px` so an opening pane
uncovers it instead of squeezing it — the same trick as the pinned collapsing blocks, and
it was there first.

## Where it stands

At 16.7ms per frame: the two rows that travel 214 and 218px peak at 27px a frame opening
and 28px closing, and both directions are monotonic to rest — the worst step against the
direction of travel is 1.8px opening and 0.6px closing, and the interior rows never step
more than 4px in either direction. The three-beat version peaked at 89px a frame with 5–7
reversals per row and was still moving at 700ms. It is 340ms now.

Those peaks were 24 and 31 at the 240px spine, over a 190px travel. Widening the spine to
300 unstacked the card's two chips onto one line, which took 24px out of the collapsed
plan's height and gave the rows below it that much further to go in the same 340ms, so the
peaks grew with the distance.

The closing figure went to 35 with a 19px reversal behind it before `ldDur` was retuned —
see *Rewrapping is not motion* above. Worth knowing how that hid: a per-row summary that
records only the worst single backward STEP called it 5.3px and looked survivable, because
the reversal was nineteen small frames rather than one big one. Sum the backward deltas per
row, or a slow drift the whole length of the tail reads as noise.

## How to measure it

To check smoothness, don't look — measure.

Click the toggle in a probe script, then `document.getAnimations()`, `pause()` them all,
and step every animation's `currentTime` together in 16.7ms increments, reading
`getBoundingClientRect().top` of each row into an attribute you can pull out with
`--dump-dom`. That gives an exact per-frame trace with no dependence on wall time. Then
diff consecutive frames: a **sign change is worse than a big step** — 60px in one frame is
a fast collapse, 20px the wrong way is judder. Total the backward deltas as well as taking
the worst one: 19 frames of 1px the wrong way is the same 19px of judder as one frame of it,
and only the total catches it.

Two gates, or the trace lies:

- Gate on the page having actually rendered (`.sc-placeholder` gone, `sc-dc-streaming` off,
  tops non-zero), or you will click a hydrating page, get zeros, and read them as real.
- Give the click a turn to render before asking for `getAnimations()`. Read it
  synchronously after `.click()` and you get zero animations and a flat, lying trace —
  and the traces come out labelled one click behind.

The resting states are the check that the machinery is invisible: screenshot both, diff
against the previous build, and expect **zero** differing pixels. `min-height: 3lh` must
equal the spine's real line count and `min-width` the open width, so if either is wrong the
diff says so immediately. A change that only touches one state — the chat width did — must
leave the other state pixel-identical.

## Traps in headless Chrome

Chrome's `--virtual-time-budget` advances layout transitions but **not** compositor
opacity, so mid-flight frames captured that way lie — they showed an empty middle that was
actually a 47%-opacity spine. To check a transition, render a static frame with hand-solved
intermediate values.

When patching the logic to force a state for such a frame, replace the *whole* initialiser
expression: swapping only `get('step') === '3'` leaves `location.search).true`, which is
legal JS evaluating to `undefined`, so the page quietly renders the state you were trying
to leave and the test passes for the wrong reason.
