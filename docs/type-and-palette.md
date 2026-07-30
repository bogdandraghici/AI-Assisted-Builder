# Type, spacing and palette

`Design System.dc.html` is the authority; this is what it comes to in practice.

## Type ladder

§04 holds the ladder — nine named roles over five sizes (10 · 12 · 14 · 18 · 24) and only
FlowX line-heights (12 · 16 · 18 · 22 · 24 · 28 · 38). Pick a role; never invent a size.
**There is no 11px and no 13px** — both existed before and made the levels
indistinguishable, which is the whole problem the ladder solves.

## Spacing

The FlowX scale (4 · 6 · 8 · 12 · 16 · 24 · 56). An off-scale number is only allowed when
it is derived optical alignment, not spacing — e.g. `padding-left: 28px` to sit under a
12px number gutter plus its 16px gap.

## Two families, and the distinction is load-bearing

Open Sans is plan language; mono is everything the product says *about* the plan —
identifiers, counts, timestamps, and every uppercase micro-label and state chip.

A micro-label is `10/12 · 400 mono` at `+.14em`, and it labels a panel or a column and
nothing else. Prose inside the elastic column is capped at 800px so a line stays readable.

## Palette

§01 and §03 — ground `#0b1218`, side pane `#0e161d`, lifted card `#131c24`, outlined card
`#101820` behind `#1c2530`, hairline `#18212a`.

**There are no elevation shadows.** The only lift is a wash of the disagreement orange plus a
tinted edge — `linear-gradient(180deg, rgba(242,118,43,.14), rgba(242,118,43,.06) 45%,
rgba(242,118,43,.025))` behind `1px solid rgba(242,118,43,.30)`, always 180°, strongest at
the top edge and thinning, but never all the way out, by the bottom. `box-shadow` is only
ever the 3px focus ring. A control standing on the wash still hovers to `#131c24` like any
other — the hover wash lifts towards the light; it never opens a hole to `#0b1218`.

**The wash means one thing: this is waiting on your reply.** Nothing else may wear it. Not a
count, not a mention, not a settled thing, not a step that merely matters. Two of them on a
screen is normal and says you owe two replies — the contested step in the plan, and the open
question in the step pane. *Which* reply is the glyph and the word, never the colour:
`ph-warning` with `Not agreed`, `ph-question-mark` with `Not answered`. Wash it where the
reply is given, so what is asked and where you answer it are the same object.

**There are no left status stripes.** A 3px stripe used to be a four-colour key — orange,
green, blue, grey for who had settled a card — which asks the reader to learn four colours to
be told what a word says plainly, and so reads as decoration, because that is all an
unlearnable key can be. Status is a mono micro-label now (`Answered by you`), and what was
open takes the wash.

**Blue is "you are here"; it never means status.** The step open beside the spine wears a
1px solid `#4d97ea` outline at `outline-offset: 3px` — the same edge the drawing's focal
node wears. An outline, not a border, so it is pure paint and no pinned measurement
moves; and not a box-shadow, which stays reserved for the focus ring. The wash keeps its
own orange edge underneath, because selection and "your move" are orthogonal and may
stack. Selection exists only in the spine state — the wide plan never shows it, because
nothing is open there.

**A previewed customer screen gets its own light palette, and it stays inside the frame.**
Ground `#ffffff`, surface `#f4f6f8`, text `#1a2129`, secondary `#5c6975`, hairline `#dfe5ea`,
action `#2b7fe4`, radius 8 — the same type ladder and the same family, so it is recognisably
built here without being mistaken for here. It is a *different product*, and the one
confusion a test screen cannot afford is the app reading as another pane of the tool. When
the tool has to speak over the app it does so in the dark palette, and that is what makes it
legible as the tool: white is the thing you built, dark is the thing that built it. The
light values never leave the frame — not into the strip, the chat or the chrome.

The frame itself is chrome, so it takes the system's own 12px radius and needs no exemption.
There is still no shadow under it — its hairline is the whole separation it needs.

**The previewed app's layout is its own, not the tool's, and not another platform's.** A web
app gets two columns where a phone would stack, a table where a phone would list, and 24/38
where a phone would use 18/28 — because a narrow column stretched across 1030px is the mobile
layout in a bigger window, which tests a layout nobody designed. The reverse holds too. What
the *run* is shown to have done stays identical either way; only the app changes.

**Two question marks, one rule.** A grey `ph-question-mark` marks a *mention* of a
question — the "N to settle" counts, the chip, the captions — and takes the colour of
the count or caption it precedes (`#8b98a5` or `#5c6975`), never one of its own. The
orange one marks the question itself, on the one card where you answer it, and nowhere
else. Same rule as the wash: the glyph says which kind, the colour only ever says
"your move".
