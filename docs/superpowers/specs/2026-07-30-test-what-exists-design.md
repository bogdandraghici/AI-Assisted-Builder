# Test what exists

Screen 02 of `Conversational Builder v2.dc.html`, settled 2026-07-30.

## What it is

`Test what exists` is the green `ph-play` in screen 01's top bar, beside
`build 2 · 09:37 · 7 resources exist`. That strapline is the premise: **two of five steps
are built**, so a run of this journey cannot finish. The screen's subject is therefore not
a preview — it is the **edge**. It has to say exactly where what exists stops, in plan
language, without pretending the app is whole.

## The premise is already in the source, and it is exact

- Top bar: `build 2 · 09:37`.
- Chat: you asked to drop the merchant check at **09:39**; the agent refused at **09:41**;
  you conceded "keep it, but the customer never waits on it" at **09:44**.
- Step 3's resource table: `merchantCheck.integrationWorkflow` — *"the check itself"* —
  **exists, build 2**. What does not exist is `enquiryToContext.dataMapper` ("carries the
  merchant's answer into step 4") and `enquiryOutcome.enum`. That is step 3's `2 not built`.

So **the thing you asked to drop already exists and already runs.** It was built at 09:37,
two minutes before you objected. A run of what exists therefore goes *through* the check,
unseen, in the 3s the drawing claims — and stops because the answer has nowhere to go.

This is what earns the screen its reason to exist. The claim under dispute is *"it slows
the customer down"*; the run is the evidence it does not. `unseen · 3s` was the agent's
drawing; now it is what happened.

**The run stops after step 3, not at it.**

## Geometry — 360 + 440 + 640 = 1440

Throughout, `⍰` stands for a grey `ph-question-mark` at 10px in the colour of the count or
caption it precedes — never a colour of its own.

### Chat, 360, and it does not move

Same width, same composer, same messages as screen 01, plus one turn. The chat is the one
thing on this prototype that never moves; screen 02 does not break that.

- New agent turn at **09:46**, ~4 lines at the pane's 278px measure (≈170 chars):
  *"I ran what exists. Both built steps, then the check — which went out while the customer
  was still typing and came back in three seconds. It stops there."*
- Composer caption changes from *"Anything you type lands on the step it is about."* to
  **"Anything you type lands on the step you are looking at."** On the plan the agent
  infers the step; here you are looking at one. This is the run's yield — what you noticed
  becomes plan language on the right step — and it is the existing promise sharpened, not
  a new mechanism.
- **Measured to fit:** blocks total 676 of 723 usable (pane 852 − 105 composer − 24 top
  padding, minus six 24px gaps). No clipping and no scroller. If the agent's turn grows
  past ~5 lines this stops being true — shorten the copy, do not add a scroller.

### Rail, 440

Headed `The plan`, at 18/28 700. Subtitle is the plan's own, verbatim:
`Two of five built · one not agreed · ⍰ four to settle`. It is the same object as the wide
plan and the spine and must not acquire a third name. The five-segment dash bar comes
across unchanged at 18px, for the same reason.

Padding `20px 24px 16px` → inner 392. Row: 20px dot gutter + 16px gap → 356px title
column. **Row titles at 14/22 600** — the plan's own title role, not the spine's
compressed 12/18: on this screen the rail *is* the plan, not a companion to a wider one.

Dot vocabulary is the spine's, unchanged: green `ph-check`, orange `ph-warning`, dashed
grey circle, solid-bordered circle with `ph-user`. All three group headings show
(`Agreed` #2ea86a, `Not agreed` #f2762b, `Next` #5c6975), plus the trailing "two more steps
will be needed" note.

Each row gains **one mono line stacked under the title, never beside it** — the counts-column
rule from `docs/motion.md`, applied for the same reason (a wider column narrows the title):

| Row | Mono line |
|-----|-----------|
| 1 | `ran · picked 1 of 12` |
| 2 | `ran · 22 words` |
| 3 | `ran · unseen · 3s` |
| 4 | `not built` |
| 5 | `not built` |

`ran` lines in `#8b98a5`; `not built` in `#5c6975`. Grey `⍰ N to settle` stays exactly
where the spine puts it, in the count's own colour.

**Two channels, no conflation:** the dot says agreement state, the mono says what the run
did. Neither is the other's colour.

**Measured to 811 of 852** — no scroller, so no act is below a fold and the pinned-act rule
does not bite. If the rail ever overflows, the act pins beneath it the way the step pane's
does.

### Stage, 640

A phone, **360 × 760**, centred — 140px of ground each side, vertically centred with ~16px
slack at 900 and more on a taller viewport. Bezel is `1px solid #26313d` at **32px radius**,
`overflow: hidden`, and **no shadow**.

The 32px radius is **hardware geometry, not the UI radius scale** — the same exemption
Phosphor's icon sizes get from the type ladder. It is the only off-scale radius on the
screen and it exists because a 360px slab at 12px does not read as a phone.

Below the phone, one mono 12/16 `#8b98a5` caption doing the job the phone cannot do for
itself:

- `at=1` → `step 1 · they picked one of twelve`
- `at=2` → `step 2 · twenty-two words, then Continue`
- `at=3` → `step 3 · the check ran behind this screen · unseen · 3s`

### The phone's palette: light, and contained

The app renders in its own light palette so it is unmistakably **not another pane of the
tool**: ground `#ffffff`, surface `#f4f6f8`, text `#1a2129`, secondary `#5c6975`, hairline
`#dfe5ea`, action `#2b7fe4` (the same FlowX blue), radius 8. Type is the same ladder and
the same family — 18/28, 14/22, 12/18. Status bar reads `09:46`.

The palette is confined to the inside of the bezel. It never touches the rail, the chat or
the chrome.

## The three stage states, and one way forward

`?at=` on `location.search`, exactly the way screen 01 reads `?step=`. Default is **3**.

1. **`at=1` — "Which payment?"** A transaction list. One row is live and tappable →
   advances to 2.
2. **`at=2` — "What happened?"** Typed text and a live `Continue` → advances to 3.
3. **`at=3` (default) — the edge.** The same screen as 2, under an `rgba(11,18,24,.72)`
   veil, with **the tool drawing over the app**: a `#101820` panel, `1px solid #26313d`,
   radius 12, padding 16, ~296px wide, centred in the frame. Mono micro-label
   `WHAT EXISTS ENDS HERE` (10/12, +.14em, `#8b98a5`), then 12/18 `#c3cdd7`: *"The customer
   would go on to send a receipt. That step is not built."* **No forward affordance**,
   because there is not one.

The panel is in the builder's dark palette on purpose: white is the thing you built, dark
is the tool speaking. It carries **no wash** — it is a fact about the build, not your move.

**Everything else inside the phone is a still** — no hover, no `cursor: pointer`, no
tabindex. The phone offers exactly what the run offers: one way on, or none. A hoverable
`Continue` on a screen you cannot drive would be the phone lying, which is the same rule
as the plan card's `pointer-events` gate.

The stage swaps **instantly, via `sc-if`** — no cross-fade. Three genuinely different
screens are not one thing changing shape, so the "one element per thing" rule does not
apply and `sc-if` is what `docs/motion.md` reserves for exactly this.

## The edge, and the act

**Step 3's rail row wears the wash** and carries `Agree` / `Comment further` — the same
pair, the same 36px, the same glyphs (`ph-check`, `ph-chat-teardrop-text`) as the plan
card and the step pane's pinned footer — plus the same caption in the same words:
`⍰ One question still to settle before I build it.`

It is the third view of one object and it says the same thing in the same words, per the
one-status-word rule. It wears the wash because **this is where you now have the best
reason to answer**: you have just watched the disputed step run without touching the
customer. Wash it where the reply is given.

The card: padding 16, radius 12, the wash gradient behind `rgba(242,118,43,.30)`. Title
(max 324px), mono `ran · unseen · 3s`, an `rgba(242,118,43,.16)` hairline, the act row,
then the caption beneath the buttons.

**Then the boundary**, between that card and the `NEXT` heading: a `#26313d` hairline with
mono `WHAT EXISTS ENDS HERE` (10/12, +.14em, `#5c6975`) and one line of plan language,
12/18 `#8b98a5`:

> The merchant's answer has nowhere to go until the evidence step exists.

Grey, never orange — it is a fact about the build, not your move. And it names **no
resource**: `enquiryToContext.dataMapper` is behind a disclosure on screen 01 and stays
there.

## Colour discipline

- **One wash on this screen**, and it is the same object the plan washes. Not the boundary,
  not the `not built` rows, not the overlay panel.
- **Blue is "you are here":** a 1px `#4d97ea` outline at `outline-offset: 3px` on the rail
  row whose screen the stage is showing. Stacks with the wash on row 3 at `at=3`, which is
  the orthogonality the rule already permits.
- Rows 4 and 5 are **inert**: `pointer-events: none`, `tabindex="-1"`, no hover. Nothing
  offers what it cannot do.
- Every `N to settle` mention keeps its grey `ph-question-mark`. No orange question mark
  appears on this screen — there is no question you can answer here.

## Top bar

- **Back arrow and the `The plan` crumb go live** (`pointer-events: auto`, `tabindex="0"`,
  `#c3cdd7`). Up from a run is the plan, and unlike the journey list it exists.
- Crumb: `Journeys / Card payment dispute / The plan / Test run`, with `Test run` white 600
  as "where you are".
- Strapline becomes the run's identity in the same slot:
  `ran build 2 · 09:46 · reached 3 of 5`.
- Green button becomes `Run again`, same `ph-play`, same `#1d9a5c`.
- Undo pair, kebab, bell, help and avatar unchanged.

## Two deliberate inertnesses

Both are unimplemented chrome, not affordances that are wrong for the current state — the
same fidelity the bell and the kebab already have:

1. `Test what exists` on screen 01 stays inert, and so does `Run again` here. The screens
   are linked by `index.html`, the way `?step=3` already is. A real `location.href` link
   would work standalone but jump you out of the source doc.
2. Nothing inside the phone except the one forward action does anything.

## Wiring

All screen blocks in the source share one `Component` and one `renderVals()`, so screen
02's keys must not collide with screen 01's.

- `state.at` joins `state.step` and `state.tech`, initialised from
  `new URLSearchParams(location.search).get('at')`, defaulting to `3`, clamped to 1–3.
- New render values namespaced away from the existing ones (`atSel1..3` for the blue
  outline, `goTo1..3` for the setters, `at1`/`at2`/`at3` booleans for the `sc-if`s).
- `Escape` stays bound to closing the step and does nothing on screen 02.
- No `min-width` pins are needed: nothing on screen 02 collapses or resizes, so no
  measurement is load-bearing across a transition.
- Screen 02's opening tag must carry `width: 1440px; height: 900px;`,
  `border: 1px solid #26313d; border-radius: 12px; ` and `flex: none; ` verbatim, or
  `build-screens.py` hard-fails when it strips the doc framing.
- Its caption block must use the exact `CAPTION_OPEN` string and its title must start
  `Screen 02 — ` so the generator can pull it.

## `index.html`

Two new sections, matching the existing two-embed pattern:

- **03** — the run at the edge (`screen-02.dc.html`, default `at=3`).
- **04** — scrubbed back to the first step (`screen-02.dc.html?at=1`).

## Verification

- `python3 build-screens.py` writes `screen-02.dc.html` and leaves `screen-01.dc.html`
  byte-identical.
- Screen 01 resting states are **pixel-identical** to the previous build in both states —
  screen 02 must not touch the shared `Component`'s existing keys or the one-beat move.
- Screen 02 at 1440×900: no scrollbar in the rail, no scrollbar in the chat, all three
  `at` states render, and the phone's bezel is not clipped.
- Keyboard: tab reaches rail rows 1–3, `Agree`, `Comment further`, the live phone action,
  the back arrow and the crumb — and reaches **nothing** in rows 4–5 or in the phone's
  stills.
- `prefers-reduced-motion` needs no new handling: nothing on screen 02 animates but hovers
  and the discrete `sc-if` swap.

## Docs to update in the same change

- `CLAUDE.md` — screen 02 in "Generated screens", and the run-as-evidence rule under "Two
  kinds of waiting" (a run of what exists may carry the wash for the step it is evidence
  about).
- `docs/type-and-palette.md` — the contained light palette for previewed customer screens,
  and the hardware-geometry exemption for the bezel radius.
