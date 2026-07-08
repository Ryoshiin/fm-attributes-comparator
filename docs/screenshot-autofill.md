# Screenshot autofill (design)

Goal: let the user upload a Football Manager player profile screenshot and have the attribute
inputs filled automatically, instead of typing 20-36 numbers by hand.

Status: implemented (v1). This doc reflects the shipped behavior; update it as it changes.

## Approach

Two engines, chosen in **Settings** (not in the import modal — the modal just shows the current engine
and a "Change in Settings" link):

- **Gemini (cloud) - primary/default.** Sends the screenshot to Google's Gemini Flash vision model,
  which returns structured JSON of the attributes. Far more accurate than OCR on small/colored FM
  digits. Uses the **free tier** (no cost for personal use). Bring-your-own-key: the user pastes their
  own free Gemini API key, stored only in their browser (`localStorage` key `fm_gemini_key`) and sent
  directly from the browser to Google. No shared key is embedded (that would be stealable), keeping
  the app serverless. Trade-off: on the free tier Google may use the images to improve their models,
  and it needs internet.
- **Offline (Tesseract.js) - fallback.** In-browser OCR (CDN, `tesseract.js@5.1.1`); screenshot never
  leaves the browser. Accuracy on small/colored digits is limited (see below).

- Flow: user clicks **Import** (in the sticky action row, or the first-run tip) -> provides an image via
  the single hero drop-zone (**Browse** button, **Paste** button, Ctrl+V, or **drag-and-drop**) ->
  **Read screenshot** -> a **review** grid lets the user fix any misreads (missing ones flagged "not
  read") -> **Apply** writes values into the chosen player's inputs. In compare mode the modal offers a
  Player 1 / Player 2 target selector (a `Seg` control). The engine/key are configured in Settings.
- The chosen engine is remembered across sessions (`localStorage` key `fm_ocr_engine`).
- **Auto-name (Gemini only):** the prompt also returns the player's `name` (and `position`); on
  Apply the detected name is written into the target slot's name field. Tesseract does not auto-name.

## Implementation (where it lives in `src/index.html`)

- `GEMINI_MODEL` / `GEMINI_PROMPT` (module-level): model id (`gemini-2.5-flash`) and the extraction
  prompt (asks for JSON `{name, position, goalkeeper, attributes}` with exact FM attribute names,
  values 1-20, no guessing). `geminiExtract(dataUrl, key)` (module-level, pure) posts the image inline
  (base64) to `generativelanguage.googleapis.com/.../:generateContent?key=...` with `responseMimeType:
  application/json`, then maps the returned names through `MAP_GK`/`MAP_OUT` to app keys and returns
  `{res, isGk, name, raw}`. `tesseractExtract(dataUrl, onProg)` is the offline counterpart returning
  the same shape (with `name:''`). Both are module-scope so `runOcr` in `App` just awaits one of them.
- Engine + key state: `ocrEngine` ('gemini' | 'tesseract', persisted via `chooseEngine` to
  `fm_ocr_engine`), `gkey` (+ `saveGkey`, persisted to `fm_gemini_key`). `runOcr` picks the extractor
  by `ocrEngine`; both paths converge on the same review UI.
- `MAP_OUT` / `MAP_GK` (module-level, next to `attr`/`imp`): on-screen FM name (lowercased) -> app
  key. Outfield and GK are separate maps because several names (Agility, Acceleration, Anticipation,
  Concentration, Positioning) map to different keys for a keeper. Used by both engines.
- `pairFuzzy(low,map)` (module-level): the core parser. Splits the OCR text into whitespace tokens and
  walks them in order, keeping a small buffer of the last 3 word-tokens. When it hits a 1-2 digit
  number it fuzzy-matches the buffered words (last 3 / 2 / 1 joined) against the attribute names and
  assigns the value to the best match, then clears the buffer. This is robust to (a) the wide,
  varying gaps between a label and its right-aligned value, (b) columns read line-major or
  column-major, and (c) small OCR misspellings of the label (e.g. "Aglity", "Tackiing"). Numbers with
  no matching label nearby (Set Pieces like Corners/Long Throws, "Goalkeeper Rating 3/10") are
  dropped.
- `lev(a,b)` + `matchName(cand,names)`: Levenshtein distance and a nearest-name matcher with an
  edit-distance threshold (1 for names <=5 chars, else 2) to keep fuzzy matching tight.
- `parseOcr(text)`: GK detection runs `pairFuzzy` over `GK_DET` (goalkeeping-EXCLUSIVE names, distinct
  keys); `>= 2` matched = keeper. Then parses values via `pairFuzzy` with `MAP_GK` or `MAP_OUT`.
- Review modal has a "Show raw OCR text (debug)" `<details>` (`ocrRaw`) to inspect what Tesseract
  produced when a value looks wrong.
- OCR runs on the original uploaded image. (A 2x canvas upscale was tried and reverted: the
  re-encode blurred the small text and made digit reads worse - e.g. 11 -> 1, 15 -> 1.)
- Review step shows the FULL expected attribute set for the detected profile (all `MAP_OUT` or
  `MAP_GK` targets), not just matches. Anything not detected is flagged `not read` (amber ring) and
  pre-filled with the target player's current value, so a missed read is visible and editable rather
  than silently left at the default. `ocrFound` holds the keys actually read from the image.
- **Soft confidence warnings.** Beyond `not read`, each *read* value is checked by `warnOf(k)` (local
  to the review render) for likely misreads and, if any, marked `check` (orange text + orange input
  ring, `title` lists the reasons); the header shows a `· N to check` count. Heuristics: (a) the read
  differs from the player's current value by ≥6, (b) an extreme value (≤2 or ≥19), and (c) tesseract
  only — a single-digit read (<10), since Tesseract often drops a leading "1" (16→6). These are
  advisory only; they never block Apply or change the value, they just draw the eye to the risky rows.
- OCR state: `showOcr`, `ocrTarget`, `ocrImg`, `ocrBusy`, `ocrProg`, `ocrRes`, `ocrGk`, `ocrErr`,
  `ocrFound`, `ocrRaw`, `ocrName` (detected name), `dragging` (drop-zone highlight), `ocrEngine`,
  `gkey`.
- Handlers: `loadImgBlob` (shared file/paste/drop loader), `onOcrFile`, `pasteClip` (async Clipboard
  API), a `useEffect` (active while the modal is open) wiring window `paste` (Ctrl+V) plus
  `dragover`/`dragleave`/`drop` for drag-and-drop, `runOcr`, `updOcr`, `applyOcr`, `closeOcr`. On
  apply, the detected name (Gemini) is written to the target slot and, if a GK was detected, the mode
  switches to `goalkeeper` (and away from it for an outfield screenshot).

## Screenshot layout (from the reference images)

FM player profiles list attributes in three labelled columns:

- **Technical** - e.g. Crossing, Dribbling, Finishing, First Touch, Heading, Long Shots, Marking,
  Passing, Tackling, Technique (plus Set Pieces: Corners, Free Kick Taking, Long Throws, Penalty).
- **Mental** - e.g. Aggression, Anticipation, Bravery, Composure, Concentration, Decisions,
  Determination, Flair, Leadership, Off The Ball, Positioning, Teamwork, Vision, Work Rate.
- **Physical** - e.g. Acceleration, Agility, Balance, Jumping Reach, Natural Fitness, Pace, Stamina,
  Strength.

Goalkeepers show a Goalkeeping column instead (Reflexes, Handling, One On Ones, etc.).

Each row is `<attribute name> <number 1-20>`. The screenshots are dark-themed with the number
right-aligned next to the label.

## Parsing notes / risks

- Map on-screen English FM names to the app's camelCase keys (e.g. "First Touch" -> `firstTouch`,
  "Work Rate" -> `workRate`, "Jumping Reach" -> `jumpingReach`). The app currently stores French
  labels in `attr`; the English->key map is separate from the display label.
- OCR can confuse digits (e.g. 1 vs 7, 13 vs 18) and merge/label lines; a manual correction UI is
  required, not optional.
- Numbers outside 1-20 are misreads -> flag or clamp.
- Not every attribute on screen is used by the impact model; extra attributes can be ignored.
- Confirm whether the GK layout is reliably distinguishable so the app can pick the right mode.

## Open questions / possible improvements

- Cropping to the attribute columns gives the cleanest reads. `pairFuzzy` handles the wide
  label-to-value gaps in a crop; full screenshots also work but carry more noise.
- `upscale` enlarges images narrower than 2400px by 3x (under 1200px) or 2x using NEAREST-NEIGHBOR
  (`imageSmoothingEnabled=false`) to make small digits bigger without blur. An earlier attempt used
  smoothing (bilinear) and made reads worse (11->1) - do not re-enable smoothing here.
- Remaining limitation: Tesseract still sometimes misreads small/colored digits outright (e.g. 17 ->
  "[4", 14 -> "I", 11 -> "1"). Non-numeric garbage is refused (flagged "not read"); a two-digit value
  read as one digit shows a wrong-but-valid number. The full-list review is the safety net - always
  eyeball the values before applying.
- `consistency` is a hidden attribute not shown on FM profiles, so it is never auto-filled.
