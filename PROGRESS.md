# PROGRESS — Absolute

## v8: the first build on template v3 (session 5, 8 Oct 2026)

John approved upgrading this repo in place (8 Oct), as a one-off. v7 is tagged `v7-final`
(`7304f82`), and that tag is the one-step rollback. v8 reaches `main` through one pull request John
merges.

### What changed from v7
- **The template is copied in,** and v7's own files are gone:
  - removed: `dataset.py`, `gen.py`, `test.js`, `deploy.sh`, `icons/favicon.png`;
  - replaced: v7's `README.md`, `index.html`, `styles.css`, `data.js`, `sw.js` and `manifest.json`;
  - `qrcode.js` and `fonts/` were byte-identical to the template's already.
- **The data** (`dataset.json`) follows the change report John approved (Research-Repo
  `sessions/2026-10-06-absolute/change-report.md`):
  - 134 rows (v7: 123): 11 added, 2 re-dated, 1 moved, none removed;
  - every date is cited to its DC Database page and revision;
  - writer and artist credits on every row.
- **Every v7 sort key is kept as its row's `id`.** Saved progress, bookmarks and old `ABSO1:` QR
  codes all line up.
- **The skins:** the signature skin "Absolute" (v7's "absolute" look, `signature.css` plus tokens
  in `dataset.json`), then Paper, Newsprint, Pull and Night. A v7 visitor's pick carries over:
  "absolute", "tabbed" and never-picked → Absolute; "classic" → Pull.
- **No workbook** (`deliverables.workbook: false`).

### Checked on this data before the pull request
- `python3 tools/verify_gate.py`: 134 rows, 0 failures, gate passed.
- `node test/run.js`: 1483 assertions, 0 failed.
- `npm run test:layout`: real Chromium, 349 checks, 0 failed (10 suites).
- **The in-place upgrade proof:** 42 of 42, in Research-Repo
  `sessions/2026-10-06-absolute/upgrade-proof/` (the script, its report and the screenshots).

### Known limit
Within 10 minutes of using v7, the first reopen of the app can still show v7. GitHub Pages lets
the browser keep v7's page that long, and v7's worker serves it. v8's worker takes over behind it,
and the next open is v8 with everything intact. After those 10 minutes it opens straight into v8.

### Open (John)
- **The signature skin couldn't carry** the big era numerals, the row-number gutter, the bracketed
  `[X]` marks, the `//` arc prefix or full-width era bars. Each one would move things, which skins
  may not do.

### Next
After the merge: verify every served file against `main` by sha256 at
https://asimkard.github.io/Absolute/, and check the removed v7 files return 404.
