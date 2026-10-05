# Modern Table: instructions for Claude sessions working in this repo

Owner: Ben (GitHub `blaframboise`). The game is served by GitHub Pages from `main` at the repo root, so whatever `index.html` is on `main` is what players get.

## Layout
- `src/app.src.html`: game source (single page; placeholders `/*__DECKS__*/[]` and `/*__PEERJS__*/` are filled by the build).
- `src/decks.txt`: built-in decklists (format below).
- `src/peerjs.min.js`: vendored PeerJS.
- `build.py`: `python3 build.py` writes `index.html`. It drops any deck with fewer than 60 main or more than 15 sideboard cards and refuses to build with fewer than 10 valid decks.
- `index.html`: generated. Never hand-edit; always rebuild and commit it together with the `src/` change.

## Versioning
`const VERSION = 'v1.NN'` in `src/app.src.html`. Bump NN by one for any feature or behaviour change. Do NOT bump it for a decks-only refresh.

## decks.txt format
```
# comment lines start with #
@@ Event name|YYYY-MM-DD|tournament id|place|Archetype|Pilot|deck id
4 Card Name
...
Sideboard
2 Card Name
```
One `N Card Name` per line, exact card names, split cards as `Wear // Tear`. No `Deck`/`Companion` label lines (a companion is already in the sideboard). No `|` inside a field. Place is `1st`, `2nd`, ...

## Weekly deck refresh (what the Monday scheduled task does)
Goal: replace `src/decks.txt` with top finishers from Modern events of the last 14 days on MTGGoldfish, rebuild, commit, push to `main`.
1. Search: WebFetch `https://www.mtggoldfish.com/tournament_searches/create?tournament_search%5Bname%5D=&tournament_search%5Bformat%5D=modern&tournament_search%5Bdate_range%5D=MM%2FDD%2FYYYY+-+MM%2FDD%2FYYYY&commit=Search` (14 days ago to today). First page only.
2. Pick up to 6 events, newest first: Challenges, Super Qualifiers, Showcases, large paper events. Skip Leagues.
3. For each, WebFetch `https://www.mtggoldfish.com/tournament/ID` for the first 8 rows: place, deck name, pilot, deck id.
4. Choose at most 36 decks: every distinct archetype once first (highest finish), then remaining top-4 finishers.
5. Fetch lists from `https://www.mtggoldfish.com/deck/arena_download/DECKID` (`/deck/download/` is robots-blocked; `/deck/ID` has no list in its HTML). ONE AT A TIME, never in parallel, at most 4 per turn. On HTTP 429 do not retry that page; stop fetching after the second refusal and use what you have. The shell cannot reach mtggoldfish; only WebFetch can.
6. Keep decks from the existing `src/decks.txt` that are still within the last 14 days and were not re-fetched, so a rate-limited week does not shrink the list. Drop decks older than 14 days only if at least 20 decks remain without them.
7. Write `src/decks.txt`, run `python3 build.py`, check its last line reports at least 10 decks and none unexpectedly dropped.
8. Commit `src/decks.txt` and `index.html` with message `Weekly deck refresh YYYY-MM-DD (N decks)` and push to `main`. If the push is refused, do not try other routes; report it.
Colour-code archetype names from MTGGoldfish (WU, UR, WBG) may be replaced with a plain name only when the list makes it obvious.

## Feedback
The game links to a Google Form ("Send feedback" on the front page and in the table menu), pre-filling the version and game details. Responses go to a Google Sheet in Ben's Drive, which Ben reviews before asking for changes. The form URL and field ids are constants near the top of `src/app.src.html` (`FEEDBACK_FORM`, `FB_VERSION`, `FB_DETAILS`).

## Testing
The page needs Scryfall and a PeerJS broker at runtime, which the cloud sandbox cannot reach. For a smoke test, load `index.html` in the pre-installed Chromium with Playwright, mock `api.scryfall.com/cards/collection`, and check the deck screen lists the decks and a solo game starts.
