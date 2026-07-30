# Spine selection and the question's mark

Two additions to `Conversational Builder v2.dc.html` (screen 01), settled 2026-07-30.

## The problem

The orange wash means one thing — *this is waiting on your reply* — but two things were
leaning on it that it does not mean:

1. **Selection.** With step 3 open, the spine's washed card reads as "the step you have
   open" only by coincidence: it is the sole highlighted row. Open any agreed step and
   nothing in the spine would mark it, and meanwhile the coincidence teaches the reader
   that orange = "where I am", which corrupts the wash's one meaning.
2. **The question kind of waiting has no vocabulary on the plan.** Step 3's card is
   washed for the *argument*, but it also announces its question, while the identical
   questions on rows 4 and 5 are bare grey counts. The reader learns "orange card =
   where the questions live". `ph-question-mark` — the mark that means question — never
   appears on the plan screen at all.

## 1. Selection: a blue "you are here" edge, spine only

**Rule:** blue edge = the step open beside you; orange wash = waiting on your reply.
Orthogonal, and they may stack. Selection exists only in the spine state — the wide plan
never shows it, because nothing is open there.

**Implementation:**

- A 1px solid `#4d97ea` **outline** with `outline-offset: 3px` on the open step's spine
  card. Outline, not border: pure paint, so no pinned `min-width` / `min-height` moves.
- The wash's own `rgba(242,118,43,.30)` border is untouched — the wash spec stays whole;
  the blue ring sits outside it with a visible gap.
- Distinct from the focus ring by construction: crisp 1px solid outline vs the soft 3px
  translucent `box-shadow`. Echoes vocabulary already on screen — the drawing's focal
  node ("Ask the merchant") carries the same `#4d97ea` edge.
- **Timing:** `outline-color` fades between `rgba(77,151,234,0)` and full on the spine
  labels' own schedule (`rmOp`: in 200ms after a 200ms delay, out 100ms with no delay),
  so the wide plan never flashes blue and nothing new travels during the beat.
- Only step 3 opens in this prototype, so only its card ever wears the ring. The rule
  generalises to any row once other steps become openable.

## 2. Questions get their mark: grey `ph-question-mark` on every mention

**Rule:** the grey question mark marks a *mention* of a question; the orange one marks
the question itself, where you answer it. Colour keeps meaning "your move" and nothing
else — same rule as the wash, applied to the glyph.

Grey `ph-question-mark` (10px, taking the same colour as the count or caption it
precedes — `#8b98a5` or `#5c6975`, never a colour of its own) goes in front of:

- Rows 4 and 5's stacked `2 to settle` / `1 to settle` mono counts — both the wide
  counts column and the spine's under-title counts.
- The spine card's `1 to settle` chip.
- The plan subtitle's "four to settle".
- The two captions "One question still to settle (before I build it)" — the plan card's
  footer and the step pane's pinned footer, plus the step pane's section note.

The orange `ph-question-mark` stays exactly where it is: on the `Not answered` card in
the step pane, and nowhere else.

## Motion guardrails

- The stacked counts likely cost **zero** width — "Nothing built yet ›" and
  "1 of 2 exists ›" are each wider than their glyph-prefixed count line — but this is
  measured, never assumed: font-loaded probe per `docs/motion.md`, then resting-state
  screenshot diff against the previous build expecting changed pixels *only* where
  glyphs were added.
- The real risk is the spine chip: `1 to settle` grows ~14px inside the pinned 206px
  flex-wrap row. If the chip pair no longer fits side by side, the collapsed plan's
  height changes and the beat's peaks move. Fallback order: trim chip padding first;
  only re-measure pinned numbers if that fails.
- Run the box-underrun trace after the change; it must stay sub-line (< 18px, currently
  ~2px).

## Docs updated in the same change

- `docs/type-and-palette.md`: the blue-edge rule and the grey/orange question-mark rule.
- `CLAUDE.md`, "Two kinds of waiting": the mention-vs-question glyph rule and that
  selection never spends orange.
