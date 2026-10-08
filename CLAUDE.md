# CLAUDE.md — Absolute (read this before touching anything)

This is the live Absolute Universe tracker, built on the Pull List template v3
(ASIMKARD/Pull-List-template) since v8. Every rule below comes from the template;
the first section is Absolute's own.

## Repo rules (read first, every session)

- **`main` is live** (GitHub Pages, https://asimkard.github.io/Absolute/). Nothing
  reaches it except through a pull request John merges, and only once it all passes
  on this repo's data: the harness, the layout suite, the verify gate and CI.
- **Never edit any repo without John's explicit say-so.** John approved upgrading
  this repo in place for the v8 pilot (8 Oct 2026), a one-off. Any later change here
  needs his say-so again. Every other repo is read-only unless he says otherwise
  (the template's `CLAUDE.md` holds the full repo-safety rule).
- **Rollback:** `v7-final` tags v7 (`7304f82`). Reverting `main` to it restores v7
  exactly; v8 never wrote v7's saved progress, so v7 reads it as it was.
- **Template changes go to the template first.** The app shell (`index.html`,
  `app.js`, `styles.css`), `tools/`, `schema/` and `test/` are the template's; fix
  them there, prove them there, then copy them here. Absolute's own files are
  `dataset.json`, `signature.css`, `icons/`, `README.md`, this file and `PROGRESS.md`.
- Each session works on its own branch and pushes it. Commit everything: sessions
  start from a fresh container. Report the **assertion count** of both suites.
  Update `PROGRESS.md` at the end of every step. Stop cleanly: every step ends
  committed, pushed and green.
- **Every date is fetched and cited, never recalled.** Each row's `date.source`
  names the DC Database page and revision. The research (the API caches, the
  change report John approved) is in Research-Repo
  `sessions/2026-10-06-absolute/`.

## The loop

```
python3 tools/build.py          # dataset.json → data.js, sw.js, manifest.json
python3 tools/verify_gate.py    # the pinned verify.py + the order the app shows
node test/run.js                # jsdom harness — report the assertion count
npm run test:layout             # real Chromium — report the count
python3 tools/build.py --check  # generated files fresh
```

## Repo map
| Path | What it is |
|---|---|
| `dataset.json` | the master — **edit this** (and `signature.css` for the look) |
| `data.js`, `sw.js`, `manifest.json` | **generated** by `tools/build.py` — never hand-edit |
| `index.html`, `app.js`, `styles.css`, `qrcode.js`, `fonts/` | the template's app shell |
| `icons/` | Absolute's own icons |
| `tools/`, `schema/`, `test/` | the template's build, gate, schema and suites |
| `README.md`, `BUILD-NOTES.md`, `MIGRATING.md`, `.claude/skills/comic-tracker-build/` | this tracker; conventions; migrating; the Skill |

## Data rules (v3)
- **`id` is the progress key and never changes.** Migrated trackers keep their old
  sort key as `id`; new trackers default `id` to `issueId`. Removing an id
  orphans someone's saved progress: the build fails unless it is listed in
  `retiredIds`.
- **`issueId` is the cross-tracker identity**: `<series>-<volume-start-year>[-<token>…]`,
  lowercase kebab, e.g. `justice-league-2016-32`, `x-men-1963-annual-1`,
  `…-half`, `…-minus-1`, `…-1-5`, `sonic-frontiers-2022`, `sonic-prime-2022-s1e3`.
  Event chapters reference it; the merge dedupes on it.
- **Sort keys are derived, never hand-written**: `RRRR·YYYYMM·NNN` (13 digits).
  Era ranks start at 5000, spaced 10 — add an earlier era with a lower rank,
  never renumber. `NNN` is assigned by the build within each era+month; `seq` is
  only a hint. `altKey` is pure publication order (no era rank).
- **Dates are sourced individually** (`date: {cover, onsale?, source}`). Never
  interpolate, never write series data from memory.
- **Events state their era** in `dataset.json` (`events: [{id, era}]`); chapters
  the tracker doesn't have are placed inside that era by date then event order.
- **Credits**: full canonical names, never surnames alone. Missing credits warn
  (coverage %), and fail only under `strictCredits`.
- **Durations (per format), whole minutes.**
  - **Comics:** every comic counts as one issue, timed by the minutes-per-issue setting.
    A comic `duration` or `durations.comic` fails the build.
  - **Shows:** `durations: {screen: 22}` gives the default, and a row's `duration` overrides it.
  - **Games:** each game needs its own `duration`. A missing one warns (coverage %), adds
    nothing to time left, and shows as "+N untimed".
  - **Finish-by** = minutes left ÷ (issues per week × minutes per issue).
- **Read fields by name, never by position.** v2's generator read columns by
  index and a missing column shifted every downstream field into garbage.
- **`dataVersion` ties a compact QR code to one list.** It hashes the ids in row
  order, and the compact QR stores marks by position. Any change to the ids or
  their order changes it, which by design invalidates old QR codes. The full
  code and the backup file are keyed on id and survive. Old v2 codes are decoded
  in v2's key order, rebuilt from legacy ids **plus `retiredIds`**: keep retired
  ids listed, or every later position shifts.

## UI rules (v3)
- **Data-driven visibility: a rule, not a one-off (decided 4 Oct).** A control or
  section renders only when the dataset gives it something to do. The data
  decides this, never a setting or a skin; skins never hide a control.
  - **One medium:** no per-format header lines, no format filter, no progress-mode
    setting, no duration copy and no "+N untimed". Verbs are that medium's own; for
    comics that is plain "Read".
  - **The rest:**
    - no Characters section without presence data or two or more strands;
    - no Creators section without credits;
    - no Essential/Complete toggle without events;
    - no ALT toggle without ALT rows;
    - no order switch without a second order;
    - no bands without periods;
    - no era jump bar with only one era.
  - **Extended (accepted 4 Oct).** Each control needs:
    - depth, type and strand chips: two or more in use;
    - "Include cameos": cameo data;
    - "Mandatory only": both mandatory and optional rows;
    - "Notes only" and tap to reveal: notes;
    - "Gap notes": a gap note;
    - the era filter, era picker, Mark range, "Newest era first" and era colours: two eras;
    - the look-up link: `searchUrl`;
    - the skin control: two or more configured skins;
    - help copy that names bands: periods.

    "+N untimed" follows the data: comics never lack a length, so a comics-only tracker never
    shows it. A saved filter for a control that isn't offered is ignored.
  - **How:** one capability map, built once at boot from the data, decides every
    case. A missing capability means the control is **not rendered**; don't hide it
    with CSS. That keeps the reachability guard and the visibility tests in
    agreement.
  - **Every new control** is added through that map, and it gets a row in the
    visibility suite.
    - The suite boots a bare fixture (comics only, one era, no extras), where none
      of these controls appear, and the full fixture, where all of them do.
    - A self-test forces each capability on in turn and must catch it.

---

## Traps carried over from v2 (each one cost real time)

Tag key: **[applies]** still true in v3 · **[superseded §N]** the v3 spec section
that removes the cause — keep the lesson anyway.

### Duplicate function names — [applies]
JavaScript keeps the **last** definition and silently discards earlier ones. v2
shipped `switchTab` twice, `jumpToIssue` three times, `issueByKey` twice: you
patch the copy that never runs. The harness fails if any function is defined
twice (found by brace-depth scan, never by a newline-brace string). Fix that
first if it fires.

### jsdom has no layout engine — [applies]
The harness can't see overlap, gaps, sticky positioning, size or truncation.
Three layout "fixes" shipped without changing anything visible because they
were reasoned about, not measured. Use real Chromium (Playwright,
`executablePath: '/opt/pw-browsers/chromium'` in cloud sessions) for anything
visual. If no browser is available, say so rather than guessing.
Session 3: the fourth tab pushed the page wider than 390 px, and at 320 px it was
cut off inside the tab bar. Both were invisible to jsdom. Sweep every tab at
320 px and check `scrollWidth`. `test/layout/10-sweep` does this on every push.
Session 4: reduced motion had never worked. A `* { transition: none }` rule has no
specificity, so every class rule that declares a transition beat it, and a test of the
CSS text "passed". Every duration now scales with `--motion`, and `test/layout/20-motion`
measures it.
After session 4 (John's phone): a depth chip was cut off inside its own scrolling row,
and Settings rows split their buttons. The page never got wider, so the sweep passed. "No
page overflow" is not "it fits": `10-sweep` now checks every control against every box
that clips it.

### grep can't see multi-line CSS selectors — [applies]
A grouped rule spanning lines won't match `grep '\.tabs.*{'`. Ask the browser:
iterate `document.styleSheets` and test `el.matches(rule.selectorText)`.
`cssRules` throws on `file://` — serve over `http://localhost`.

### position:sticky vs position:relative — [applies]
`top:` means "stick here" on sticky and "shift down" on relative. One skin
declaration of `position:relative` turned every sticky offset into a
displacement: a phantom gap, an overlap and floating text.

### content-visibility — [superseded §1, resolved session 4]
v2 removed `content-visibility:auto` + `contain-intrinsic-size`: blank unpainted
rows on iOS, mis-positioned scroll-to-row, wrong estimates. Spec §1 says keep
the performance guard. **Closed (John, 4 Oct):** v3 lands
collapsed and renders an era's rows only when it is first expanded (90-render:
5,000 rows render none at landing). The guard is unnecessary, and the iOS bugs
can't occur. Don't add `content-visibility`.

### Two stores for one setting — [superseded §2 by design; lesson applies]
v2 split settings between `state.settings` and `view`; a control that wrote one
while `applyView()` read the other appeared to work, then snapped back.
v3 has **one** settings store. Write to the store the reader uses.

### Writes are debounced 400 ms — [applies]
A change made just before close never reaches storage unless flushed. Flush on
`pagehide` and `visibilitychange` (hidden). When testing persistence, **wait for
the debounce** — reading 30 ms early looks exactly like a broken save and has
produced two false bug reports.
In a real browser, seed storage with `addInitScript` **before** load. If you write
localStorage while the page is open and then reload, the `pagehide` flush correctly
saves the app's in-memory state over what you wrote, which looks like the app
ignoring your seed.

### One guard word is also a CSS keyword — [applies]
The franchise-string guard matches "absolute" (the Absolute line), and CSS
needs `position: absolute`. The guard exempts that one declaration, with a
self-test that "Absolute Batman" is still caught. If a guard fires on
legitimate code, narrow it with a self-test. Never rename real code to dodge
it, and never weaken it wholesale.

### Maps compared as JSON depend on key order — [applies]
`t.eq` compares `JSON.stringify` output, and decoders return maps in row order.
Compare canonicalised (keys sorted) when the order is not the point, or a
correct round-trip reads as a failure.

### One toast, one Undo — [applies]
Bulk marks, Clear all, Replace and preset deletes each offer Undo on the
toast. A newer toast replaces it, so only the latest action is undoable.
Write tests (and expectations) one action at a time.

### Colour lived in five places — [superseded §1: one token block]
v2: base `:root`, seven `data-bg` swatches, the signature-skin palette, the
`--eN-t/-a/-d` era ramp per scheme, and `.era[data-e=N]` rules that stopped at
era 25 while the ramp went to 34 — rows past the lowest ceiling went unstyled.
v3: one colour-token block; ramps generated, no cap (60+ eras), and a guard
asserts exactly one token block. Era tints are pale washes.
**Design dark skins against the surface.** The first dark skin had 39 of 54
era colours failing WCAG AA (worst 1.35:1). Measure contrast; don't eyeball.
**How v3 holds this (session 4):**
- `:root` holds numeric inputs (hue, saturation, lightness per role) and derives
  every colour from them. Skins, paper swatches and the other Look settings set
  inputs only, in bare `:root[data-…]` blocks, and 80-guards rejects anything else.
- Outside `:root` the only colours allowed are *derived* ones, where every component
  is a token: an era's wash is `oklch(var(--era-tint-l) var(--era-tint-c) var(--era-h))`,
  with `--era-h` computed from the era's index (`style="--ei:N"`, a token, not a
  style). A literal component anywhere is caught.
- `test/layout/40-look` measures contrast for every skin × paper and for 64 eras.
  Its first run caught Night's control outlines at 2.95:1 on the warm and mint
  papers: at the same HSL lightness, a yellowish hue is brighter.
- Every control must be at least 24 × 24 px (WCAG 2.5.8). Creator names inside a
  line of credits are the inline exception.

### A control that saves a value nobody reads — [applies]
v2's refresh reminder stored an interval for a whole build before anything used
it. Add the setting and the code that acts on it in the same change, or not at all.

### iOS specifics — [applies]
- A home-screen app registers its **own** service worker; it must be opened once
  **online** after install. Settings → Offline reports readiness.
- iOS caches the home-screen icon at install; delete and re-add to change it.
- `respondWith(undefined)` is a network error: an offline fallback that misses
  its cache gives a blank page. Always end with a real `Response`.

### Service worker — [applies]
- Evaluate `sw.js`, don't just parse it: a temporal-dead-zone error passes
  `node --check` and silently kills the worker. The harness runs it in a vm.
- Network-first for the shell (`skipWaiting` + `clients.claim`), cache-first
  only for fonts and icons. A cache-first shell makes correct fixes invisible.
- `addAll` rejects the whole install on one 404 — every precached path must exist.
- Playwright's `setOffline` doesn't reach the service worker's own fetches, so an
  "offline" test can quietly pass through the server. Take the test server down
  instead (`test/layout/lib.js` `down()`), as `test/layout/70-pwa` does.
- Every tracker on one GitHub Pages site shares one cache storage. A worker
  deletes only its own old caches (`<key>-v<N>`, `<key>-<12 hex>`). v2's deleted
  everyone's.

### GitHub Pages caches every file for 10 minutes — [applies]
Pages sends `max-age=600`. The Absolute upgrade proof (session 5) caught three
consequences, and a test server that sends `no-store` had hidden all of them:
- **A new worker can precache the old deploy.** `addAll` goes through the
  browser's HTTP cache, so the worker stored v7's files under v8's cache name.
  It now precaches with `cache: 'reload'` and fetches the shell with
  `cache: 'no-cache'` (a 304 when nothing changed).
- **A reload can mix two builds.** Within those minutes Chromium takes scripts
  and styles from its memory cache, and the worker never sees the request. v8's
  `app.js` met v7's `data.js` and crashed. `app.js` now checks the data's shape
  and the stylesheet's `--skin-ok` before anything reads them. On a mismatch it
  loads both again at `?r=<time>` and starts over, once; if they're still stale
  it says so and never loops.
- **Within those minutes, the first reopen can still show the old build.** The
  old worker serves its own page from the HTTP cache. The new worker takes over
  behind it, and the next open is the new build. Nothing a new worker does can
  reach that page; `WindowClient.navigate()` closes it in headless Chromium, so
  it isn't used.

Test with `serve(dir, { pages: true })` (`test/layout/lib.js`), which sends
Pages' headers. `page.waitForFunction` doesn't wait for an async check: its
promise counts as true at once. Use `poll()` from the same file.

### Preloading isn't free — [applies]
A preloaded font competes for bandwidth with the render-blocking stylesheet. On a
1.6 Mbps link, preloading the display font cost about 100 ms of first paint, and
landed it before that paint, so the title never swaps. Preloading the body font as
well cost another 260 ms. Only the display font is preloaded; `test/layout/80-paint`
measures it. Measure before adding a preload.

### Cache name and build tag — [superseded: automatic in v3]
v2 required bumping `CACHE` and the build tag together by hand. v3 derives the
cache name from a content hash at build time, and the readable version ("v13")
counts up by itself (session 5). Still: check the version on the phone before
judging anything; Settings → About shows the hash.

### A crashed harness looks like a pass — [applies]
A crash mid-suite reports fewer passes, not a failure. `test/run.js` fails on
any suite crash and on **zero assertions**; always report the count.

### Test the real rendered DOM — [applies]
Never synthetic elements. Boot the real `index.html` + generated `data.js`.

## Before declaring anything done
1. `python3 tools/build.py --check` — generated files are fresh.
2. `python3 tools/verify_gate.py` — gate passed.
3. `node test/run.js` and `npm run test:layout` — 0 failures, counts reported.
4. Manifest and icons are Absolute's own.
5. After a merge, verify the deploy **by hash**, every file, not HTTP 200; GitHub
   Pages takes ~100 s.
