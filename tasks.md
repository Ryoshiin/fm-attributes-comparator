# Tasks

Backlog + change log for this project. Group tasks by topic. Keep the "Done / notes" log dated so
history lives in the repo, not in chat.

Status keys: `[ ]` todo, `[~]` in progress, `[x]` done.

## Format

- A task is one line: `- [ ] <title> - <enough inline context to act without re-deriving it>`.
- Group related tasks under a `### <topic>` heading.
- Finish a task: set it to `[x]` and add a dated entry under "Done / notes".
- A "Done / notes" entry: `- YYYY-MM-DD: <what changed and why; any gotcha; link to docs/ if useful>`.

### Paused work (`[~]`)

When you stop mid-task, set it to `[~]` and add an indented note so the next session resumes safely:

```
- [~] <title>
  - Context: <what this task is and why>
  - Current state: <what is done so far; branch; files touched>
  - Next step: <the very next action to take>
  - Validation: <command / check to confirm it works, and the last result>
  - Risks/blockers: <anything unresolved that could break>
```

## Setup

- [x] Run the `define-project` skill to fill in the `AGENTS.md` Project section and `README.md`.

## Backlog

### Screenshot autofill (new feature)

- [x] Add a cloud vision engine (Gemini Flash, free tier, bring-your-own-key) as the primary reader,
  keeping Tesseract as an offline fallback, to fix OCR inconsistency on small/colored digits.

- [x] Add an upload control to load an FM player profile screenshot into the app.
- [x] Integrate Tesseract.js (CDN) to OCR the screenshot in-browser.
- [x] Parse OCR output: map on-screen FM attribute names to the camelCase keys in `attr`, read the
  number next to each, clamp to 1-20. Cover Technical / Mental / Physical and GK layouts.
- [x] Show a review/correction step so the user can fix misreads before applying values to the inputs.
- [x] Decide + implement whether autofill targets Player 1, Player 2, or a chosen slot.
- [x] Document the final approach and known accuracy limits in `docs/screenshot-autofill.md`.
- [x] Make misses recoverable: review shows the full expected attribute set, flagging any "not read".
- [x] Fix parser to handle wide label-to-value gaps (cropped columns): pair each label with the next
  number in reading order instead of a fixed char window.

### Translate UI to English

- [x] Replace French labels/help/tierlist text with English throughout `src/index.html`. Keep the
  attribute keys unchanged; only change display strings. FM's English attribute names are the target.

### UI/UX overhaul

- [x] Hoist input components to module scope (fix `AttrIn` focus loss) and split the mega top card into
  a header + a sticky results bar with primary actions and live Points/GF/GA.
- [x] Add a "why this score" breakdown panel (per-attribute contribution bars; P1−P2 delta in compare).
- [x] Comparison diff view: both values per row with the higher side highlighted + delta, a "biggest
  differences" strip, and an "only differences" toggle.
- [x] Named players + saved roster (`fm_roster`): inline name fields, save/load into P1/P2, delete,
  copy/swap/per-slot reset, and Gemini auto-name on import.
- [x] Screenshot flow polish: promote Import to a primary action, drag-and-drop, first-run tip
  (`fm_seen_tip`), persist engine choice (`fm_ocr_engine`).
- [x] Visual refresh: tier-colored input borders, consistent spacing/rounded cards, transitions,
  responsive layout. See `docs/ui.md`.
- [x] Minimal + dense pass: remove all icons/emoji, reduce to a single amber accent (green/red kept
  only where meaningful), and tighten spacing + masonry grid so the default view fits without scrolling.
- [x] Rebalance after the too-bland minimal pass: line-icons back on a single top action row (labels
  hide below md), a Settings modal (API key, OCR engine, theme, reset, help, tiers), and colour back
  (tier-coloured letters + per-category accent colours). Web-searched FM26/dashboard UI for direction.
- [x] Polish pass: rename to "FM Attribute Comparator", fix inconsistent category card outlines,
  simplify the Import modal, make the compare Δ the focal metric with a verdict, drop the redundant
  "Biggest differences" strip (keep the "only differences" filter), icon-ify History/Players delete,
  clean footer, harmonise the Settings modal (shared `Seg` segmented control).
- [x] Comparison Focus Rework: Extract metrics into a large Hero Scoreboard, redesign `AttrRowCompare`
  to a center-aligned Tug-of-war layout, simplify the sticky header, and improve typography/spacing.

### Docs

- [ ] Fill `docs/impact-model.md` with the exact scoring formula, the mode subsets, and the weight source.

- 2026-07-07: Implemented the "UI Rework and Comparison Focus" plan. Extracted metrics and name inputs
  from the sticky header into a new, prominent `metricSingle` / `metricCompare` Hero Scoreboard component.
  Redesigned `AttrRowCompare` to use a center-aligned Tug-of-war layout (P1 value left, attribute name
  center, P2 value right) with inline delta highlights. Simplified the sticky header to just the title
  and action buttons. Increased typography sizes (e.g., `text-[10px]` -> `text-xs`, `text-xs` -> `text-sm`)
  and padding across all components (`Cat`, `Breakdown`, panels) for a cleaner, less cramped look.
  Adjusted the `Cat` grid layout to `xl:columns-2` in compare mode to accommodate the wider rows.
- 2026-07-08: Published the project. Created the GitHub repo `Ryoshiin/fm-attributes-comparator`
  (initial commit), then set it **public** and added GitHub Pages hosting via a GitHub Actions workflow
  (`.github/workflows/deploy-pages.yml`) that uploads `src/` as the Pages artifact (app root, since
  `index.html` lives in `src/`). Pages `build_type=workflow` had to be enabled once via the API by the
  repo owner because the Actions `GITHUB_TOKEN` can't create the Pages site ("Resource not accessible by
  integration"); after that the deploy succeeds and every push to `master` auto-publishes. Live at
  https://ryoshiin.github.io/fm-attributes-comparator/ (verified HTTP 200). Public hosting is safe: no
  server secrets; the Gemini key is user-entered and stored only in the browser. Updated README (live
  URL, corrected shipped-vs-planned features, deploy note).
- 2026-07-07: Layout consistency tweaks on `src/index.html`. **Removed the big bottom gap** in All mode
  (short Hidden column sitting far below Physical): replaced the row-major CSS grid with a JS-balanced
  flex masonry — `ncol` from a tracked `vw` at the same breakpoints, then categories greedily packed into
  the shortest column by attribute count (`outCats`/`catCols`). No reflow on value edits (packing depends
  only on visible counts) and short categories pack under tall ones. **Consistent goalkeeper width**: GK
  now renders in the same `1/ncol` grid (`gkGrid`) instead of `max-w-sm`, so its column matches the
  outfield columns when switching modes. **Steadied the compare scoreboard**: converted it to a fixed
  `md:grid-cols-3` with `tabular-nums` everywhere and fixed-width delta cells (`dStat`, `w-12`), so the
  layout no longer shifts as numbers change; toned point totals down to match the deltas (`text-2xl`;
  single view also reduced). **Reset buttons moved to the outer edge** next to each player (P1 left, P2
  right) instead of by the centre. Docs/ui.md updated. Verified served (HTTP 200).
- 2026-07-07: Bugfix + polish pass on `src/index.html` (on top of user's manual restyle). **Fixed the
  category cards jumping left/right** when editing values: root cause was the CSS multi-column masonry
  (`columns-*`), which rebalances content across columns on any height change — replaced with a stable
  CSS grid (`grid grid-cols-*` + `items-start`). Removed the "AI-looking" category left-accent bar and
  header dot; `Cat` cards now use a plain box border with a category-coloured uppercase title over a
  bottom divider (dropped the `hex` from `CAT`). **Reworked the compare scoreboard**: removed the
  box-in-box (the inner Δ card border) in favour of three balanced columns split by vertical dividers
  (`border-x`), and toned down the oversized point totals (5xl→3xl) so P1/Δ/P2 are harmonised; kept
  names, points, Pts/GF/GA deltas and verdict. Removed the confusing compare action row (Copy P1→P2 /
  Swap dropped; per-slot **Reset moved to a small icon next to each player name**, single view too via
  new `IcReset`); the **Only differences** toggle moved to the right of the mode-tabs row. Changed the
  History **load** icon from a download glyph to a log-in/enter glyph (`IcLoad`). **Settings**: replaced
  the two `<details>` dropdowns (How to use, Attribute tiers) with always-visible static sections so it
  no longer mixes buttons + dropdowns. Removed now-unused `copy12`/`swap`. Docs/ui.md updated. Verified.
- 2026-07-07: UI polish pass on `src/index.html` per feedback. Renamed app to **FM Attribute
  Comparator** (`<title>` + `<h1>`). Fixed inconsistent category outlines: `CAT` full-border colour
  classes were being dropped for some categories by Tailwind CDN ordering, so cards now use a visible
  gray box border (`th.cb` bumped to gray-700/300) plus a deterministic 2px coloured left accent via
  inline `borderLeftColor` (hex per category). Simplified the **Import modal**: single hero dropzone
  (Browse/Paste/drag/Ctrl-V, preview inline), compare target as a `Seg`, engine+key demoted to one
  small "Change in Settings" line (engine buttons removed from the modal). Made the compare metric the
  focal point: a highlighted amber `Δ · Pts GF GA` block with a plain-language verdict ("X is better"
  / "Evenly matched"), P1/P2 totals de-emphasised. Removed the redundant "Biggest differences" strip
  (and `topDiffs`); moved the "Only differences" toggle into the compare actions row. History/Players
  delete are now trash icons (`IcTrash`); History load is a `IcLoad` icon. Cleaned the footer to one
  centred line with a top divider. Harmonised the Settings modal into labelled rows using a new shared
  `Seg` segmented control (theme, engine) + `IcLoad`. Logic unchanged. Verified served (HTTP 200).
- 2026-07-07: Rebalanced the UI after feedback that the fully-minimal pass was too bland / removed too
  much. Web-searched FM26 + comparison-dashboard UI for direction (tile/card grouping, accent colours,
  percentile-bar comparison, consolidated navbar). Changes to `src/index.html`: reintroduced small
  line-icons (`Ic*` on a shared `Svg` wrapper) on a SINGLE top action row (labels hide below `md` so it
  never wraps to two rows); added a **Settings modal** (gear) that consolidates theme, OCR engine,
  Gemini API key, reset-all, and collapsible Help + Attribute-tiers reference (removed the old header
  Light/Dark/Help/Tiers buttons and `st`/`sh` state, added `showSettings`); brought colour back with
  intent — tier-coloured attribute letters (`TIER_TEXT`) and per-category accent colours (`CAT`:
  emerald/sky/amber/violet/rose) as a left border + dot + header on each `Cat` card. Logic unchanged.
  Docs/ui.md + AGENTS.md convention updated.
- 2026-07-07: Minimal + dense restyle of `src/index.html` per user request ("fit the full page without
  scrolling; more minimalist UI with fewer icons/colors"). Removed all SVG/emoji icons (text labels
  only), collapsed the palette to a single amber accent (`th.acc`; no gradients) with green/red kept
  only where they encode meaning (metrics, compare deltas, breakdown bars, diff chips — `POS`/`NEG`/
  `ACC` constants). Tiers now show as a muted gray letter (no colored row borders); the S-D colors
  live only in the Tiers panel. Densified: compact rows/steppers, smaller buttons/headings, and a
  masonry attribute grid (`columns-*` + `break-inside-avoid`, up to 3 cols single / 2 compare) so the
  default view fits without scrolling. Logic (`imp`/`calcImpact`, OCR) unchanged. `th` is now a flat
  dark/light object. Docs/ui.md updated. Note: true no-scroll depends on viewport height; All mode is
  the tallest.
- 2026-07-07: UI/UX overhaul of `src/index.html` (single file, no build). Rewrote the render around
  module-scope components (`Num`, `AttrRowSingle`, `AttrRowCompare`, `Cat`, `Bar`, `Breakdown`) so
  attribute inputs no longer remount and lose focus (root cause: `AttrIn` was redefined every render).
  Added a **sticky results bar** (name fields + Import/Compare/Save/Breakdown/Players/History/Reset +
  live Points/GF/GA, or `P1·Δ·P2` in compare). New features: **breakdown** panel (contribution bars,
  `contribsOf`/`deltaContribs`), **diff view** (both values per row, higher side ringed, biggest-
  differences strip, `onlyDiff` toggle), **named players + roster** (`fm_roster`; save/load into
  P1/P2, delete, copy/swap/per-slot reset), **screenshot polish** (primary Import CTA, drag-and-drop,
  first-run tip `fm_seen_tip`, engine persisted to `fm_ocr_engine`, Gemini auto-name via extended
  prompt), and a **visual refresh** (tier-colored row borders, rounded cards, transitions, responsive
  compare stacking). Impact math (`imp`/`calcImpact`) and OCR parsing unchanged. Moved `runGemini` ->
  module `geminiExtract` and added `tesseractExtract`. New keys: `fm_roster`, `fm_ocr_engine`,
  `fm_seen_tip`. New doc `docs/ui.md`; `docs/screenshot-autofill.md` updated. Verified served at
  http://localhost:8000 (HTTP 200); JSX transpiles in-browser (no local Babel available to compile-check).
- 2026-07-07: Added Google Gemini Flash (`gemini-2.5-flash`) as the primary, more accurate screenshot
  reader; kept Tesseract as an offline fallback (engine toggle in the modal). Bring-your-own free API
  key (`gkey`/`saveGkey`, stored in `localStorage` as `fm_gemini_key`), posted directly from the
  browser to `generativelanguage.googleapis.com` with `responseMimeType: application/json`. `runGemini`
  parses the returned `{goalkeeper, attributes}` JSON through `MAP_GK`/`MAP_OUT`. Chosen because
  Gemini's free tier costs nothing for personal use and reads small/colored FM digits far better than
  Tesseract, which was too inconsistent. No shared key embedded (would be stealable); keeps app
  serverless. Constants `GEMINI_MODEL`/`GEMINI_PROMPT` added; `runOcr` now branches on `ocrEngine`.
- 2026-07-07: Added nearest-neighbor upscaling (`upscale`, imageSmoothing off) to enlarge small
  digits without blur, targeting number misreads (17->"[4", 14->"I", 11->"1", 10->"[0]" seen in raw
  OCR). Distinct from the earlier reverted bilinear/smoothing upscale, which blurred text. Tuned to 3x
  for <1200px wide, 2x for <2400px. Accepted limitation: generic OCR still misreads the odd small
  colored digit; the full-list review with "not read" flags is the safety net.
- 2026-07-07: Made attribute-name matching fuzzy to fix labels still missed on crops (Agility,
  Tackling) - OCR slightly misspells some labels, which exact matching skipped. New `pairFuzzy`
  tokenizes the OCR text, buffers recent words, and on each number fuzzy-matches (Levenshtein `lev` +
  `matchName`, edit-distance threshold 1 for short names else 2) the buffered 1-3 words to the
  nearest attribute name. Added a "Show raw OCR text (debug)" details panel (`ocrRaw`) in the review
  to diagnose bad reads. Replaced the exact-regex `pairInOrder`.
- 2026-07-07: Rewrote the OCR parser to fix labels being skipped on cropped columns (Agility,
  Strength, Tackling). Root cause: the old per-name regex only accepted the value within 6 chars of
  the label, but cropped columns have a wide gap to the right-aligned number. New `pairInOrder`
  matches all mapped labels and 1-2 digit numbers in one pass and pairs each label with the next
  number in reading order (works line-major or column-major; ignores Set Pieces / "Goalkeeper Rating"
  numbers since those labels aren't mapped). GK detection now also uses `pairInOrder` over a
  gk-exclusive `GK_DET` set (>=2 = keeper), so it is gap-robust and won't flag outfielders. Removed
  the now-unused `readVal`.
- 2026-07-07: Reverted the 2x upscale - it blurred small text and made digit reads worse (e.g. 11->1,
  15->1, on the Eric Martel screenshot). OCR runs on the original image again. Kept the full-list
  review as the safety net. Real accuracy fix (crop / word-bbox row pairing) left as a backlog item.
- 2026-07-07: Reworked the review to list the FULL expected attribute set for the detected profile,
  flagging any `not read` (amber) and pre-filling the target player's current value so misses are
  visible/editable. Added `ocrFound` state; removed the now-unused per-row remove button. (Also tried
  2x upscaling here, since reverted - see next entry.)
- 2026-07-07: Fixed GK false-positive and added clipboard paste. GK detection now counts
  goalkeeping-only attribute names that have a number next to them (>=2 = keeper) instead of matching
  the word anywhere - "Media Handling Style" in the Info panel was triggering GK mode on outfield
  players. Added `readVal` helper, `loadImgBlob`, `pasteClip` (Clipboard API 📋 Paste button) and a
  window `paste` listener (Ctrl+V) active while the modal is open.
- 2026-07-07: Implemented screenshot autofill (v1) in `src/index.html`. Added Tesseract.js CDN
  (`@5.1.1`), module-level `MAP_OUT`/`MAP_GK` name->key maps, `parseOcr` (GK detection + word-boundary
  regex to read the number after each attribute name, clamped 1-20), OCR state/handlers, a blue
  "📷 Screenshot" button in the impact panel, and a review modal (file upload, in-browser OCR with
  progress bar, editable/removable parsed values, P1/P2 target selector in compare mode, Apply merges
  into the chosen player). Apply auto-switches mode to `goalkeeper` when a keeper is detected. OCR is
  client-side (screenshot not uploaded); Tesseract fetches its wasm/lang from CDN on first run. Not
  yet tested against real screenshots - accuracy tuning is a follow-up. See docs/screenshot-autofill.md.
- 2026-07-07: Translated all UI text in `src/index.html` from French to English. Changed only
  display strings, not attribute keys or logic: `attr` labels -> FM English names, tier list, help
  panel, mode buttons (All/Offensive/Defensive/Goalkeeper), impact readout (P1/P2, Points, GF, GA,
  Goals For/Against), history panel (History/Load/Saved player/"P1 advantage vs P2", badges GK),
  reset button, category titles (Physical/Mental/Technical/Hidden/Goalkeeper), footer, `<title>`,
  and `lang="en"`. Also switched date/time locale `fr-FR` -> `en-GB` and sort locale `fr` -> `en`.
  Gotcha: two edits briefly dropped the `a` from footer `<a>` tags; both fixed and verified.
- 2026-07-07: Ran define-project. Archetype = code-app (single-file static web app, no build).
  Moved the existing app from `F:\Workspace\FM - Calculateur attributs - AlexJeuVideal.html` into
  `src/index.html`. Filled AGENTS.md Project section and README. Decisions: in-browser OCR via
  Tesseract.js, keep it a single self-contained HTML file (CDN React+Tailwind, no npm/build), and
  migrate all UI text from French to English. Added `docs/impact-model.md` and
  `docs/screenshot-autofill.md` stubs. Note: original French source file was moved (not copied).
