# Reading-order tracker

A static, offline-capable comics reading-order tracker. Edit the config block in
`gen.py`, generate `data.js`, deploy to GitHub Pages.

Full setup guide: `README-NEXT-FRANCHISE.md`. Project rules for Claude Code
sessions: `CLAUDE.md`.

## Files

| File | Purpose |
|---|---|
| `gen.py` | Builds `data.js`. The `FRANCHISE` block at the top is the only franchise-specific code |
| `data.js` | The whole checklist as `window.TRACKER_DATA` — generated, never hand-edited |
| `index.html` | App shell, markup and all logic |
| `styles.css` | Structure plus per-skin blocks scoped to `data-skin` |
| `sw.js` | Service worker. Network-first for the app shell, cache-first for fonts |
| `test.js` | Test harness — `node test.js` |
| `manifest.json` | PWA metadata |
| `deploy.sh` | Pushes to GitHub Pages using `GITHUB_TOKEN` from the environment |

## First run

1. Edit `FRANCHISE` in `gen.py`. **`key` must be unique across every tracker you
   host** — it namespaces localStorage and the QR sync prefix, and all your
   trackers share one origin on GitHub Pages. Reusing a key overwrites another
   tracker's saved progress.
2. Optionally define `PERIODS` and `ELSE_STORIES`; both may stay empty.
3. `python3 gen.py`
4. `node test.js` — everything should pass.
5. Update `manifest.json` (name, short_name, description, theme_color) and the
   `CACHE` string in `sw.js`.

## Deploying

**Bump `CACHE` in `sw.js` on every deploy.** The service worker keys its cache on
that string; without a change the browser keeps serving the old shell and your
fix appears to do nothing. The build tag in the header is there so you can
confirm at a glance which version is live.

## Stored state

All keys are prefixed with `<franchise key>:v1:` —

- `…:progress` — `{p: {sortKey: state}, b: [bookmarked sortKeys]}`
- `…:settings` — theme, era scheme, layout, collapsed sections, view options
- `…:filters` — depth, types, strands, era, search, ALT visibility
- `…:reviews` — `{sortKey: {r: stars, t: text}}`

Progress lives in the browser only. There is no account and no server. QR sync
moves progress between devices; codes carry a per-franchise prefix so a code
from one tracker cannot be imported into another.
