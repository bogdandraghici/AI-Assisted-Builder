# Notes

Measured values, traps, and rules for `Conversational Builder v2.dc.html`. Everything here
cost time to find. Read the relevant section before changing what it covers; add to it only
when a number changes or a new trap bites.

---

## Motion — mechanism

Motion is CSS transitions over state-driven inline styles (`style="flex-basis: {{ planBasis }}"`);
`support.js` re-resolves style strings on every render, so this animates.

- **Never `sc-if` anything that animates** — it unmounts, and an unmounted element has no
  value to transition from. `sc-if` is for things that just appear.
- **One element per thing, never two that cross-fade.** The wide plan and the 300px spine are
  the *same* five rows changing type size, dot size and padding.
- **Collapse with `grid-template-rows: 1fr → 0fr`** (`-columns` horizontally) on a wrapper whose
  child is `overflow: hidden; min-height: 0`. Closed size is then the content's own height —
  no magic number to keep in sync, which `max-height` needs.
- **Fade before you close, fade after you open.** `exOp*` run shorter than `ex*` for this.
- **The move is one beat** — everything starts at the same instant, one curve, one duration,
  both directions. Three beats peaked at 89px/frame with 5–7 reversals per row.

Two things make one beat possible:

- **Pin collapsing content so it clips instead of reflowing.** A `1fr → 0fr` box's height is a
  fraction of its *content's* height, so anything that rewraps mid-collapse grows while it
  should shrink. `min-width` = the width it has when open.
- **Let the measure outrun the pane.** A title's `max-width` travels 800 → 224 while its
  available space travels 879 → 224, so on the same curve `max-width` is always the smaller
  and it alone decides line breaks.
- **Reserve the line box before the text needs it** — `min-height: 3lh`/`2lh` animating from
  `1lh`, *leading* the measure opening, *lagging* it closing. Rewrapping is a stack of 18px
  jumps and no curve smooths a jump.
- **Whatever has no counterpart in the other state still has a SIZE, and the size is not late.**
  Group headings and per-step counts only fade, but their 76px opens with everything else.
- **No stagger on arrivals.** One fade, no translate — the pane opening is the motion.

## Motion — measured values

| Thing | Value |
|---|---|
| Easing (move) | `cubic-bezier(0.2, 0, 0.2, 1)`, 340ms |
| Easing (hover) | same, 160ms |
| Easing (lead/lag: measure + reserved box) | `cubic-bezier(0, 0, 0.4, 0.8)` |
| `ldDur` closing | **240ms**, not 340 — see below |
| Chat pane | fixed 360px, both states, takes no part in the move |
| Step pane content | `min-width: 780px` — pane uncovers it rather than squeezing it |
| Row `min-width` pins @1440 | **838 / 838 / 860 / 979 / 860 / 860 / 1015** |
| Title `max-width` final (row) | `300 − 32 pad − 8 dot − 12 gap − 24 gap = 224` |
| Title `max-width` final (card) | `248 − 2 border − 20 pad − 24 gap − 8 gap = 194` |
| `tlA` / `tlB` reservations | 3 / 2 / 3 / 2 / 2 |
| Spine's real title line counts | 2 / 1 / 2 / 2 / 2 (deliberately under the reservation) |
| `planPinH` | 816 / 812 |
| Box-underrun at rest | 1.4px opening, 2.0px closing |

The seven pins are a function of the chat pane's width and of what the counts column holds —
**anything that moves either invalidates all seven.** They shifted 80px when the chat settled
on 360, and moved again when the counts column became a single settle tag. The fourth (979)
is pinned on the contested step's **card itself**, not its contents: it has an edge the
contents would burst through before the clipping parent caught them.

`max-width` finals must include gaps — a collapsed sibling still occupies its flex `gap`. The
first attempt was 24px too generous and bought one last rewrap on the final frame.

**Over-reserving is the safe direction.** A box larger than its text can never gain a line
mid-flight, which is the whole property the pins exist for. Tightening the two loose
reservations would take 36px out of the collapsed plan's height and hand the rows below that
much further to travel in the same 340ms — a motion change, not a copy change.

**Adding a count to a plan row is not free.** It widens the counts column, narrowing the title
beside it at every instant, and a title whose real width drops below its own animating
`max-width` hands wrapping back to the pane. Keep that column as narrow as the row's state can
be said in.

**Why `ldDur` closes at 240.** At 340 on an ease that finished late, the panes rested around
300ms while 32px of reserved box per title was still collapsing — rows that had travelled 218px
down drifted **19px back up** over the last 100ms. A long, late reversal is the one thing this
move may not do. 240ms puts the shrink back inside the window where the pane is still moving:
backward drift 19.4 → 1.4px, peak forward step 35 → 28px.

### Current state of the trace

**The two travelling rows moved to 237 and 241px when the contested step's card came out from
around it and went under it, and the stepped trace has not been re-run since.** Peaks track
distance, so expect ~27–31px/frame; confirm with the stepped trace before trusting older
figures. What *was* re-measured on this build: zero width-overflow on every pinned box in both
resting states, zero height-overflow on the plan's box, `planPinH` 816 / 812, all five titles
one line at 800 in the wide plan, spine line counts 2 / 1 / 2 / 2 / 2.

Historic peaks, for the distance relation: 24/26 per frame over a 196/200px travel;
27/28 over 214/218; 24/31 over 190 at the 240px spine.

**Absolute figures are rig-specific.** A probe stepping `getAnimations()` `currentTime` reads
per-row opening back-totals up to ~5.5px on an unchanged build with zero variance across runs.
The gate is a same-rig before/after comparison — this build against the last known-good — never
these prose numbers taken cold. Box-underrun is the one figure that reproduces across rigs.

## Motion — how to measure it

Don't look, measure. Click the toggle in a probe, then `document.getAnimations()`, `pause()`
them all, step every animation's `currentTime` together in 16.7ms increments, reading
`getBoundingClientRect().top` into an attribute you pull out with `--dump-dom`.

Diff consecutive frames: **a sign change is worse than a big step.** 60px in one frame is a
fast collapse; 20px the wrong way is judder. **Total the backward deltas as well as taking the
worst one** — 19 frames of 1px is the same 19px of judder as one frame of it, and only the
total catches it. A summary recording only the worst single backward step called a 19px
reversal 5.3px and looked survivable.

**Box-underrun trace:** at every frame take `scrollHeight − min-height` over the five titles,
keep the worst. Sub-line (~2px) means no title gained a line mid-flight. Past 18px, the jumps
are back and the duration is too short.

### Traps

- **Gate on hydration — all three signals**, not two: `.sc-placeholder` gone, `sc-dc-streaming`
  off, tops non-zero. Placeholders go *before* the bindings resolve, so a probe checking only
  for them reads raw markup — `style="width: {{ chatW }}"`, invalid CSS, every rect zero — and
  a naive assertion prints PASS on it. This produced a confident, wrong bug report that
  `support.js` does not interpolate `tabindex`/`title`/`class`. It interpolates all three.
- Gate on the **target element's own** `style` attribute being free of `{{`, never on
  `document.body.innerHTML`, which contains the probe's own braces.
- **Make the probe fail loudly on impossible geometry.** A check that prints PASS over all-zero
  rects is worse than no check.
- **Give the click a turn to render** before asking for `getAnimations()`. Read it synchronously
  after `.click()` and you get zero animations, a flat lying trace, and labels one click behind.
- **Measure with webfonts loaded** — `document.fonts.ready` plus an explicit `fonts.load()` of
  the weights the plan uses. The tell is rows with no counts column staying stable while the
  others move.
- **To read the pins, lift every `min-width` to 0 at once**, read what each box is given, put
  them back. A pin can only ever report itself. Take the **floor** of what comes back — a pin
  above the real width overflows its clipping parent.
- **A reserved line box hides its own line count.** `scrollHeight` can never fall below the
  element's own height, so a title under `min-height: 3lh` reports three lines whether it needs
  three or one. Count actual line boxes: `createRange()`, `selectNodeContents`, then collapse
  `getClientRects()` by `top`. A clone with the floor lifted is not the fix — at 206/224px it
  lands on the wrong side of a borderline wrap.
- **Headless Chrome:** `--virtual-time-budget` advances layout transitions but **not** compositor
  opacity, so mid-flight frames captured that way lie. Stepped traces are unavailable there at
  all — Chrome settles transitions before `getAnimations()` is asked. Instead **check both ends**:
  transitions disabled, toggle to each resting state, take `scrollHeight − offsetHeight` over
  every pinned box. No box smaller than its content at either end ⇒ none can gain a line between.
- **When patching logic to force a state, replace the whole initialiser expression.** Swapping
  only `get('step') === '3'` leaves `location.search).true` — legal JS evaluating to `undefined`,
  so the page renders the state you were trying to leave and the test passes for the wrong reason.

### Copy changes get the same gate, and it is cheap

Four things, readable in one pass per resting state: every description still one line at its own
`min-width`; every title still one line at 800px in the wide plan; no pinned box with a non-zero
underrun; `planPinH` unmoved at 816 / 812. If all four hold, the move is untouched.

Screenshot both resting states and diff against the previous build: expect **zero** differing
pixels, or — when copy changed — differing rows confined to bands one line-height tall. A band
taller than a line, or one below the last thing you edited, is layout shifting, not text. A
change touching only one state must leave the other pixel-identical.

---

## The two kinds of waiting

- **The plan is being negotiated.** Word `Not agreed` → `Agreed`. Glyph `ph-warning`. Argued in
  the chat, settled in the step's footer.
- **A step needs a detail before it can be built.** Word `N to settle` / `Not answered`. Glyph
  `ph-question-mark` — *not* `ph-question`, which is the top bar's help button. Answered in place.

**They never gate each other.** Agreeing is never gated on answering — agreed-and-still-owing is
a legal state, so a contested step must be able to reach it. The two acts of an argument are
**agree** and **comment further**; *build* is the agent's move and belongs in no button here.
Both kinds are counted in both views, in their own units — the argument in steps, questions in
questions.

### The wash

**The wash means exactly one thing: this is waiting on your reply.** Which reply comes from the
glyph and the word, never the colour. Only two objects may carry it — the argument about the
contested step, and the one open question in the step pane. Two washes on screen is right.

**The wash is on the argument, not on the step.** In the wide plan the contested step is an
ordinary row and the washed card sits *under* it. In the spine that card is inside the same
collapse as every other row's description, so the wash falls back onto the step's own block.

**The spine's wash carries its own `Not agreed` eyebrow** (on `rmRows`/`rmOp`) — a wash with no
word beside it leaves colour saying which kind of waiting this is, which colour may never do.
It is wrapped with the title in a `gap: 0` box: a 0fr grid still pays its column's 2px gap,
which would push that one title 2px below its own dot.

**A run of what exists may carry the wash for the step it is evidence about** — screen 02 washes
step 3 in its rail and carries `Agree` / `Comment further` there. Wash it where the reply is given.

**The act is pinned, never scrolled.** The `Agree` / `Comment further` row lives outside the
scroller, same as the ×. If that row grows, the scroller's `bottom` changes with it.

### Grouping and counts

**`Built` / `Not built yet`, and there is no third heading.** `Built` is the strict word:
finished, agreed, standing. The contested step is not under it. The run strip's left heading is
`Ran`, not `Built` — the one place two views use different words on purpose, because a run can
only claim what *exists*, and all three cards under it say `ran · …`.

**One tag per plan row, the whole of the row's right-hand column, all five rows** — `N to settle`
or `Nothing to settle`, nothing else. The plan is read for what is still owed.

**The settle tag is the one mention that wears orange, and it wears a flat tint, never the wash.**
Orange when something is owed, grey when nothing is.

**The header's clauses do not partition the five steps.** `Two of five built · one not agreed ·
four to settle` is three axes; step 3 sits on all three at once. No bucket can say that.

**Orange is rationed to:** the settle tags, step 3's dot, the wash, and step 3's own eyebrow. Not
the group headings. **The contested step's chips stay grey in the spine** — there its block is
already washed under its own `Not agreed`, so a third orange thing is the two kinds of waiting
wearing one colour. In the wide plan the row carries no wash, so its tag is orange like the rest.

**Selection never spends orange.** Blue is "you are here", orange is "your move", and they stack.

### Full screen is the chat's one exception

The chat is a fixed 360 everywhere else. Full screen collapses the chat and run strip together
on the shared 340ms curve; everything the collapse passes over is pinned — the chat's two blocks
at `min-width: 360px`, the strip's five titles at `min-height: 3lh`, its boundary sentence at
`2lh`. Verified both-ends: five titles at a 54px box, boundary sentence at 36px, underrun 0.0px
at both 1032 and 1392.

---

## Affordances

Both states are always mounted, so a control that cannot act in this state must be **withdrawn
in the logic**. This is about affordances wrong for the *current state*, not unimplemented
chrome — the bell, kebab and undo pair hover and do nothing, and that is the fidelity of a
design doc.

- **Gate with `pointer-events`, in the interpolated `style` attribute.** `style-hover` /
  `style-active` / `style-focus` are passed to `pseudoClass()` verbatim (`support.js:428`) and
  take **no** `{{ }}`, so a hover can only be withdrawn by taking the pointer off the element.
- **Withdraw the `tabindex` too** — and the rule runs both ways: never leave a pointer target the
  keyboard cannot reach. A whole clipped pane does both at once via `visibility` on the pane (it
  inherits, and being discrete it cannot judder). Delay the hide by the content's fade-out.
- **`{{ … }}` resolves in *every* attribute.** `collectProps` runs `compileAttr` over all of them
  (`support.js:441`) — `tabindex`, `title`, `class` interpolate exactly like `style`. The
  `style-*` trio is the only exception. So a state-dependent tab stop, tooltip or glyph is **one
  element with an interpolated attribute**, never two behind an `sc-if`.

**Three glyphs, three meanings:** `caret-right` discloses in the pane you are already in;
`arrow-right` changes what you are looking at; `arrows-out`/`arrows-in` changes how *much* of it.
A control and its row carry the same one, and the expand pair holds the same slot in both states.

**The four ways out of a step are one control at four sizes** — ×, Esc, breadcrumb and the back
arrow share `exitTab` / `upPE` and arrive and leave together. In the plan state all of them go
quiet, because up from the plan is a journey list this prototype does not have.

**One status word per state, across both views.** A step is `Not agreed` on the wide plan's card
heading, on its spine eyebrow and in the step badge. Two views of one object naming it
differently read as a change of subject.

---

## Type, spacing and palette

`Design System.dc.html` is the authority; this is what it comes to in practice.

- **Type ladder** (§04): nine named roles over five sizes (10 · 12 · 14 · 18 · 24), FlowX
  line-heights only (12 · 16 · 18 · 22 · 24 · 28 · 38). Pick a role; never invent a size.
  **There is no 11px and no 13px.**
- **Spacing:** 4 · 6 · 8 · 12 · 16 · 24 · 56. Off-scale only for derived optical alignment
  (e.g. `padding-left: 28px` to sit under a 12px number gutter plus its 16px gap).
- **Two families, load-bearing.** Open Sans is plan language; mono is everything the product says
  *about* the plan — identifiers, counts, timestamps, uppercase micro-labels, state chips. A
  micro-label is `10/12 · 400 mono` at `+.14em`. Prose caps at 800px.

**Palette** (§01, §03): ground `#0b1218`, side pane `#0e161d`, lifted card `#131c24`, outlined
card `#101820` behind `#1c2530`, hairline `#18212a`.

- **No elevation shadows.** The only lift is the wash: `linear-gradient(180deg,
  rgba(242,118,43,.14), rgba(242,118,43,.06) 45%, rgba(242,118,43,.025))` behind `1px solid
  rgba(242,118,43,.30)` — always 180°, strongest at the top, thinning but never out. `box-shadow`
  is only ever the 3px focus ring. A control on the wash still hovers to `#131c24`; the hover
  lifts towards the light, never opening a hole to `#0b1218`.
- **The wash is the *gradient*.** The settle tag's flat tint at the same values is a different
  object: `rgba(242,118,43,.14)` behind `1px solid rgba(242,118,43,.30)` when owed,
  `#131c24` behind `#26313d` when not.
- **No left status stripes.** A four-colour key asks the reader to learn four colours to be told
  what a word says plainly. Status is a mono micro-label (`Answered by you`); what is open takes
  the wash.
- **Blue is "you are here"; it never means status.** 1px solid `#4d97ea` at `outline-offset: 3px`
  — an outline, not a border, so it is pure paint and no pinned measurement moves; not a
  box-shadow, which is reserved for the focus ring. Selection exists only in the spine state.
- **One question mark, one rule.** `ph-question-mark` marks a question wherever it appears and
  never has a colour of its own — it takes the colour of whatever it precedes (`#8b98a5`,
  `#5c6975`, or `#f2762b` inside an owed tag).

**The previewed customer screen has its own light palette, and it stays inside the frame.**
Ground `#ffffff`, surface `#f4f6f8`, text `#1a2129`, secondary `#5c6975`, hairline `#dfe5ea`,
action `#2b7fe4`, radius 8 — same ladder, same family. White is the thing you built, dark is the
thing that built it; when the tool speaks over the app it does so in the dark palette. Light
values never leave the frame. **The app's layout is its own** — a web app gets two columns where
a phone would stack, and 24/38 where a phone would use 18/28. The frame itself is chrome, so it
takes the system's 12px radius and needs no shadow.

---

## Build and preview

```
python3 build-screens.py                          # regenerate screen-NN.dc.html
python3 -m http.server 8765 --bind 127.0.0.1      # preview — always over HTTP
```

`file://` breaks `support.js` and the local fonts. Open Sans ships from `fonts/`; Phosphor icons
come from a CDN, so icons need network.

- **Two variants.** Standalone = the page filling the viewport, no framing. Embedded = inside
  `index.html`, which owns the title and description. Design size 1440×900; standalone holds a
  `min-width`/`min-height` there and scrolls below it rather than breaking.
- **Screens are generated.** `build-screens.py` splits each top-level `data-screen-label` block
  out of `Conversational Builder v2.dc.html`, stripping the doc framing. Edit the source, never
  `screen-*.dc.html` — they are overwritten and carry a `GENERATED` banner. Add a screen → re-run,
  then add its `index.html` section by hand. Remove one → re-run and its page is deleted (only
  banner-carrying pages are swept). The script hard-fails rather than emitting a broken page.
- **One page shows two states.** A two-state screen is **one** `data-screen-label` block;
  `index.html` embeds it twice, the second time as `screen-01.dc.html?step=3`, read off
  `location.search`. Two blocks would drift — that is how the chat panes misaligned once.
- **Publishing.** Repo <https://github.com/bogdandraghici/AI-Assisted-Builder> is **public**,
  branch `main`; Pages serves the root at <https://bogdandraghici.github.io/AI-Assisted-Builder/>,
  redeploying ~20s after every push. A push publishes to a URL that gets indexed — hence the
  explicit-ask rule in CLAUDE.md.
