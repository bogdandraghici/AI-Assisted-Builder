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

**There are no elevation shadows.** The one exception per screen is carried by a wash of the
disagreement orange plus a tinted edge — always 180°, strongest at the top edge and
thinning, but never all the way out, by the bottom. `box-shadow` is only ever the 3px focus
ring.

**A 3px left stripe marks the one open thing, and nothing else.** It used to be a key —
orange, green, blue, grey for who had settled a card — which asks the reader to learn four
colours in order to be told something a word says plainly, and reads as decoration, because
that is all an unlearnable key can be. Status is a mono micro-label now (`Answered by you`),
so the one stripe left on a screen is the orange one on whatever is still open.
