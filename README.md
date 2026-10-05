# Modern Table

A free-form, two-player Magic: The Gathering table that runs in the browser. Nothing is enforced; you play it like paper, and the table does the chores.

**Play:** https://blaframboise.github.io/modern-table/

One player opens a table and sends the six-letter code; the other joins with it. Card images and rules text come from Scryfall. Built-in decklists are recent Modern tournament results from MTGGoldfish, refreshed weekly.

## How this repository works

- `index.html` is the whole game in one file. It is generated; do not edit it by hand.
- `src/app.src.html` is the game's source.
- `src/decks.txt` is the built-in decklists.
- `src/peerjs.min.js` is the PeerJS library (MIT), inlined at build time for player-to-player connections.
- `python3 build.py` rebuilds `index.html` from `src/`.

Version numbers look like v1.13. The second number goes up when features change. A decks-only refresh keeps the version.

Unofficial fan project. Magic: The Gathering is a trademark of Wizards of the Coast; this project is not affiliated with or endorsed by them.
