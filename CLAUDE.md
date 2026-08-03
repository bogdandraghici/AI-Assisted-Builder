# AI Assisted Builder

The FlowX AI-assisted builder prototype. `index.html` is the screen — edit it directly, there
is no build and nothing generates it; every state is a query string on it. `Design System.dc.html`
and `Choice Cards.dc.html` are reference docs. All are static HTML rendered client-side by
`support.js`, which supplies `<x-dc>`, `sc-for`, the `style-*` pseudo-class attributes,
`DCLogic` and React.

## Working here

- This is a design prototype. Make the change and look at it. No specs, no plans, no
write-ups — prose about the change is not part of the change.
- **That rule covers the side doors too**, which is where it keeps getting broken: commit
messages are one line, comments name a trap rather than narrate the change, `docs/notes.md`
takes measured numbers and traps and never a story, and no new `.md` file appears unless I
ask for one.
- **Don't report back at length.** I read the diff and look at the screen. Say what changed
in a line or two, plus anything that is actually still broken.
- Preview over HTTP: `python3 -m http.server 8765 --bind 127.0.0.1`. Opening `file://` breaks
`support.js` and the local fonts.
- **Never push unless I explicitly ask** — the repo is public and gets indexed. Commit freely;
prefer `git revert` over rewriting history.
