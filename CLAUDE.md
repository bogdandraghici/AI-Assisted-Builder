# AI Assisted Builder

Design-doc prototypes for the FlowX AI-assisted builder. Pages are `.dc.html` — static HTML
rendered client-side by `support.js`, which supplies `<x-dc>`, `sc-for`, the `style-*`
pseudo-class attributes, `DCLogic` and React.

## Principles

- **Two variants.** Standalone fills the viewport with no doc framing; embedded is the same
  page inside `index.html`, which owns the title and description. Design size 1440×900.
- **States, not screens.** One `data-screen-label` block per screen however many states it
  has — `index.html` embeds that one page once per state.
- **Screens are generated.** Edit the source doc, never `screen-*.dc.html`.
- **One beat.** The two states are joined by a single transition, it is the most delicate thing
  in the repo, and its numbers are measured rather than derived.
- **Nothing offers what it cannot do.** Both states are always mounted, so a control that
  cannot act in this state is withdrawn in the logic — pointer and keyboard together.
- **Two kinds of waiting, and they never gate each other.** A plan being negotiated is not a
  step owing a detail, and agreeing is never gated on answering.
- **Building is the third thing owed, and it waits on both of the others.** One test at two
  grains: a step builds when it wears no wash, the plan when nothing anywhere does. Blue is the
  commit — Apply, Agree, Build; green is the machine — built, running.
- **The wash means exactly one thing: this is waiting on your reply.** Which reply it is comes
  from the glyph and the word, never the colour. Selection is blue.
- **Nothing appears or vanishes where you were looking.** A card that moves between lists is
  mounted at both ends for one beat: the slot it left closes while the slot it lands in opens,
  and what lands wears the commit's blue until the beat is over.
- **One status word per state, across both views.**
- **Pick a role from the type ladder; never invent a size.** No elevation shadows.

## Working here

- This is a design prototype with a handful of screens. Make the change, run the build, look
  at it. No specs, no plans, no write-ups — prose about the change is not part of the change.
- Preview over HTTP: `python3 -m http.server 8765 --bind 127.0.0.1`. Opening `file://` breaks
  `support.js` and the local fonts.
- **Never push unless I explicitly ask** — the repo is public and gets indexed. Commit freely;
  prefer `git revert` over rewriting history.
