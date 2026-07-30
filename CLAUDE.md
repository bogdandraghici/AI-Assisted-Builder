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
- **The wash means exactly one thing: this is waiting on your reply.** Which reply it is comes
  from the glyph and the word, never the colour. Selection is blue.
- **One status word per state, across both views.**
- **Pick a role from the type ladder; never invent a size.** No elevation shadows.

## Working here

- This is a design prototype. Bias to speed — make the change, run the build, look at it.
  Prose about the change is not part of the change.
- **Don't document much, and don't read much.** [docs/notes.md](docs/notes.md) holds the numbers
  that cost real measurement. Open it when you are about to touch the transition or a pin;
  otherwise go straight to the source doc. Write back only when a measured number changes or a
  new trap bites — a line or two, never the reasoning that got there. Same for comments in the
  source doc: state the value, don't justify it.
- Specs and plans only for large features — anything that restructures a screen or touches the
  transition. Tweaks, copy and single-state changes go straight to the source doc.
- Preview over HTTP: `python3 -m http.server 8765 --bind 127.0.0.1`. Opening `file://` breaks
  `support.js` and the local fonts.
- **Never push unless I explicitly ask** — the repo is public and gets indexed. Commit freely;
  prefer `git revert` over rewriting history.
