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

## Geometry — 360 + 1080 = 1440, and why the phone had to go

The first build put a 360px phone in a 640px stage beside a 440px rail. That was not an
aesthetic choice — 360 was the only viewport that fits there at **1:1**. A desktop viewport
wants ≥1024, and no rail width buys it:

| Rail | Stage | 1:1 desktop? |
|---|---|---|
| 440 | 640 | no |
| 300 (the spine's width) | 780 | no |
| 200 (too narrow for titles) | 880 | no |
| 0 | 1080 | yes |

**At 1440, a 1:1 desktop viewport and any vertical rail are mutually exclusive.** Scaling is
not a way out — a 1280 window at 0.5 puts the app's 14px text at 7px, a test you cannot read
— and neither is clipping, which hides the control you are meant to have used.

So the rail lies down, and that is the better shape. A run reads left to right, so the
boundary becomes a **wall across its path** rather than a rule beneath it.

### Chat, 360, and it does not move

Unchanged from the first build: same width, same composer, same messages, plus the agent's
09:46 run report (~4 lines at the pane's 278px measure) and the composer caption
**"Anything you type lands on the step you are looking at."** Measured to fit at 676 of 723
usable, no scroller. If the agent's turn grows past ~5 lines, shorten the copy.

### Stage, 1080

`padding: 24px` → inner **1032**, a column with the browser frame at `flex: 1` and the run
strip at `flex: none`. That ordering matters: a node title that wraps to a third line costs
the app height rather than clipping the strip, so nothing needs a pinned measurement.

**The frame** is `1032 × flex`, `1px solid #26313d`, radius **12** — the system's own, because
a browser frame is chrome. The 32px "hardware geometry" exemption the phone bezel needed is
gone, and the repo has no off-scale radius again. Still no shadow. Inside: a 36px chrome bar
(`#f4f6f8`, three 8px `#dfe5ea` dots, a 420px URL pill in mono) then the app.

### The run strip

Five nodes at `flex: 1 1 0px` over 1016, with a 16px wall column after node 3. The group
labels row (`2 : 1 : 2`) and the boundary row (`3 : 2`) share that denominator, so every row
lands on the same column edges without a grid.

Each node: a 20px dot on the rail with its connector running right (solid for 1–3, dashed for
4–5), then a 12px-padded title block at **12/18 600** and the run's mono line beneath it.
Titles at 12/18 rather than the plan's 14/22 — 203px of column will not carry the larger role.

| Node | Mono |
|---|---|
| 1 | `ran · picked 1 of 12` |
| 2 | `ran · 22 words` |
| 3 | `ran · unseen · 3s` |
| 4 | `not built` + `⍰ 2 to settle` |
| 5 | `not built` + `⍰ 1 to settle` |

**The wall** is a 2px solid `#26313d` vertical rule at `left: calc((100% - 16px) * 0.6)`,
running from the labels row to just above the act. Same colour as the connectors and
unmistakable anyway: they run *with* the journey, it runs *across*.

**The act is pinned under the strip** — buttons left, caption right — which is the step pane's
own footer shape, adopted for the step pane's own reason. A 203px node cannot hold a 250px
button pair, and the wash, the `⚠` and the caption are what tie the act to node 3, exactly as
the step pane's pinned footer ties to its `Not agreed` badge 470px above it.

### The app's palette and layout are both its own

Palette confined inside the frame: ground `#ffffff`, surface `#f4f6f8`, text `#1a2129`,
secondary `#5c6975`, hairline `#dfe5ea`, action `#2b7fe4`, radius 8. Same type ladder, same
family.

**And the layout is a web app's, not the phone's stretched.** Two columns where the phone
stacked, a table where it listed, 24/38 where it used 18/28 — because a narrow column
stretched across 1030px is the mobile layout in a bigger window, which tests a layout nobody
designed. What the *run* is shown to have done is identical either way; only the app changes.

The app's own header is `Dispute a payment` / `Save and exit`, which invents no brand.

## The three stage states, and one way forward

`?at=` off `location.search`, default **3**. The URL in the chrome bar replaces the caption
the bezel needed — and at the edge it still reads `what-happened`, which is the run never
having advanced, said by the app rather than about it.

1. **`at=1` — "Which payment?"** `bank.example/dispute/which-payment`. A table: merchant /
   date / card / amount, six rows plus `Showing 12 payments from the last ninety days.` Six,
   not seven — the footer is what makes `picked 1 of 12` checkable rather than trusted, and at
   seven rows the frame clipped it. Row 1 is live (`#f4f6f8`, `ph-arrow-right` in `#2b7fe4`)
   and advances to 2.
2. **`at=2` — "What happened?"** `bank.example/dispute/what-happened`. Form left (max 620,
   textarea + a 180px `Continue`), the payment's summary card right (300). `Continue` advances
   to 3.
3. **`at=3` (default) — the edge.** The same screen under an `rgba(11, 18, 24, 0.28)` veil
   (measured: `#BBBDBF` over white — light enough that it still reads as a white app), with
   the tool's plate at **bottom-left**, aligned to the app's own content column. A form leaves
   its lower half empty, so the tool speaks into the app's own emptiness, which is the better
   argument anyway. `#101820`, `1px solid #26313d`, radius 12, max-width 480:
   `WHAT EXISTS ENDS HERE` then *"The customer would go on to send a receipt. That step is not
   built."* No forward affordance, because there is not one.

**Everything else in the app is a still** — no hover, no cursor, no tabindex. The live
controls carry `tabindex="0"` and a focus ring, because the mouse can reach them and the rule
runs both ways: withdraw the tabindex with the pointer, but never leave a pointer target the
keyboard cannot reach.

The stage swaps **instantly via `sc-if`** — three genuinely different screens are not one
thing changing shape, so the "one element per thing" rule does not apply.

## The app is dense, but only as dense as the plan licenses

The app is a real enterprise web app: a brand header with the journey name, `Save and exit`,
help and the customer's own avatar in the app's greys (never the builder's blue `LC` — two
different people looking at two different things); an account strip; a progress indicator; a
scrolling body; and a sticky action bar.

**But it gains no capability the plan does not state.** Step 1 says *"Last ninety days. Card
and account already known, so nothing is typed twice."* So there is no search box and no date
picker anywhere in it — those would be the preview inventing features the agent never built,
which is a preview that lies. The account strip is that sentence made visible: facts, not
fields, with no cursor and nothing to change. Step 2 says *"Free text, then we sort it into
one of the nine reasons the scheme accepts"* — so free text only, and no reason picker. Step 4
is not built, so nothing in the app mentions a receipt.

Two things are licensed and worth having:

- **The progress indicator shows two steps and nothing after.** That is the app itself
  showing the edge, before the tool says a word. It cannot be confused with the run strip:
  one is inside the white app and counts what the customer sees, the other is the tool's own
  chrome and counts what the plan contains.
- **"Your card number is never shared with the merchant."** That is the plan's
  `cardPan.piiGuard` line — *"that one you cannot switch off"* — said to the customer instead
  of about them, naming no resource.

**The app scrolls, because a browser window does.** That is also what makes twelve table rows
honest: the run says `picked 1 of 12` and all twelve are there to be counted. The app's one
live control sits in a **sticky action bar outside the scroller**, for the same reason the
tool's act is pinned under the strip and the step pane's under its body — three panes, one
shape, and the live control never scrolls out of reach.

The typed answer is **111 characters and 22 words**, and the strip says `22 words`. It was 24
when the character counter arrived and the strip caught it. Two views of one run must agree on
the arithmetic or neither can be trusted.

## Full screen, and the chat's one exception

An `arrows-out` / `arrows-in` pair in the top bar, in the same slot in both states so the way
back is never somewhere the way in was not. Plus `Esc`, which takes full screen before it
takes an open step — full screen is the more modal of the two, and on the run screen there is
no step to close. `?full=1` puts `index.html` section 05 straight into it.

**It is a real move on the shared 340ms curve, not an `sc-if`** — an unmounted pane has no
width to travel from. The chat's `width` goes 360 → 0 with its `border-right-width`, the strip
collapses `1fr → 0fr`, and the frame takes what is freed: **1032 × 491 → 1392 × 804**.

This is the one place the chat moves, and the reason is that you asked to see only the app.
The act goes with the strip, which is honest rather than lossy: it is clipped to nothing with
`visibility` on the pane — so it is not offered, not merely hidden — and Esc brings it back.

**Everything the collapse passes over is pinned so it clips instead of reflowing**, which is
what lets it be one beat:

| Pinned | Value | Why |
|---|---|---|
| Chat's two blocks | `min-width: 360px` | every message would otherwise rewrap the whole way down |
| Strip's five titles | `min-height: 3lh` | nodes go 203 → 275px wide, so titles rewrap 3 lines → 2 |
| Boundary sentence | `min-height: 2lh` | same, at 406 → 549px |

The app needs no pin at all: every column in it is fixed or capped (`max-width: 620`, a 300px
sidebar, fixed table columns, a 480px plate), so a wider frame moves nothing.

The 3lh pin also fixes something that was merely untidy — the five mono run-lines were ragged,
sitting at different heights under titles of different depths. They align now.

## The edge, and the act

**Node 3 wears the wash**, and the act pinned beneath the strip carries `Agree` /
`Comment further` — the same pair, the same 36px, the same glyphs (`ph-check`,
`ph-chat-teardrop-text`) as the plan card and the step pane's pinned footer — plus the same
caption in the same words: `⍰ One question still to settle before I build it.`

It is the third view of one object and it says the same thing in the same words, per the
one-status-word rule. It wears the wash because **this is where you now have the best
reason to answer**: you have just watched the disputed step run without touching the
customer. Wash it where the reply is given.

Node 3's card: padding 12, radius 12, the wash gradient behind `rgba(242,118,43,.30)`, the
title and `ran · unseen · 3s`. The other four nodes carry `1px solid transparent` so all five
titles start on the same line.

**Then the boundary**, under the region it describes and starting at the wall: mono
`WHAT EXISTS ENDS HERE` (10/12, +.14em, `#5c6975`) stacked over one line of plan language,
12/18 `#8b98a5`:

> The merchant's answer has nowhere to go until the evidence step exists. Two more steps after
> these are not written yet.

The second sentence is the wide plan's trailing note, which had nowhere to go when the rail
lay down — and this is the one region on the screen that is about what does not exist, so it
belongs here rather than nowhere.

Grey, never orange — it is a fact about the build, not your move. And it names **no
resource**: `enquiryToContext.dataMapper` is behind a disclosure on screen 01 and stays
there.

## Colour discipline

- **One wash on this screen**, and it is the same object the plan washes. Not the boundary,
  not the `not built` rows, not the overlay panel.
- **Blue is "you are here":** a 1px `#4d97ea` outline at `outline-offset: 3px` on the node
  whose screen the stage is showing. Stacks with the wash on node 3 at `at=3`, which is the
  orthogonality the rule already permits.
- Nodes 4 and 5 are **inert**: `pointer-events: none`, `tabindex="-1"`, no hover. Nothing
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
2. Nothing inside the app except the one forward action does anything.

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

## What `support.js` will and will not interpolate

**`{{ … }}` resolves in `style` and in `onClick`. It does not resolve in `class`, `title` or
`tabindex`** — those render the template text verbatim. For `tabindex` that means an invalid
value and an element that is never focusable in *either* state.

So every state-dependent glyph, tooltip and tab stop on screen 02 is **two elements behind an
`sc-if`**, which is legal precisely because none of them animate: the expand/collapse pair,
and `Continue` live-vs-dead. Where the value never actually varied — the back arrow, the
crumb, the two unbuilt nodes — it is a literal, which cannot silently fail.

This was found by probing the rendered DOM, and it is **pre-existing in screen 01**: three
`tabindex="{{ exitTab }}"`, three `tabindex="{{ cardTab }}"` and two `title="{{ upTitle }}"`
have never worked there. Screen 01 is untouched by this change; the fix is its own.

## Verification

- `python3 build-screens.py` writes `screen-02.dc.html` and leaves `screen-01.dc.html`
  byte-identical.
- Screen 01 resting states are **pixel-identical** to the previous build in both states —
  screen 02 must not touch the shared `Component`'s existing keys or the one-beat move.
- Screen 02 at 1440×900: no scrollbar in the chat, all three `at` states render, the browser
  frame is not clipped, and no app content is cut off by the frame's `overflow: hidden` —
  the table's footer is the one that goes first.
- Keyboard: tab reaches strip nodes 1–3, `Agree`, `Comment further`, the app's one live
  control, the back arrow and the crumb — and reaches **nothing** in nodes 4–5 or in the
  app's stills.
- `prefers-reduced-motion` needs no new handling: nothing on screen 02 animates but hovers
  and the discrete `sc-if` swap.

## Docs to update in the same change

- `CLAUDE.md` — screen 02 in "Generated screens", and the run-as-evidence rule under "Two
  kinds of waiting" (a run of what exists may carry the wash for the step it is evidence
  about).
- `docs/type-and-palette.md` — the contained light palette for previewed customer screens,
  that the frame is chrome and takes the system's 12px radius, and that the previewed app's
  layout is its own platform's rather than the tool's or another platform's.
