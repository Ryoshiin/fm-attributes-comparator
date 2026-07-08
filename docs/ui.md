# UI / layout

How `src/index.html` is structured after the UI/UX overhaul. Single file, React + Tailwind via CDN,
no build step. This doc is the current-state reference; update it when the layout changes.

## Component structure

To avoid the focus-loss bug (a component redefined every render remounts its `<input>` and drops
focus mid-typing), all components that contain attribute inputs live at **module scope** and receive
`th` (the theme object) and handlers as props:

- `Num` - the −/number/+ stepper (clamped 1-20). Optional `ring` prop adds a highlight.
- `AttrRowSingle` - one attribute row (label + tier letter + `Num`), left border tinted by tier.
- `AttrRowCompare` - one attribute row for two players: uses a center-aligned "Tug-of-war" layout with P1 value on the left, attribute name in the center, and P2 value on the right, plus inline deltas. Used in compare mode so both players share a row.
- `Cat` - a category card (Physical / Mental / Technical / Hidden / Goalkeeper) rendering the rows;
  respects `compare` and `onlyDiff` (hides identical rows). Clean card: plain box border (`th.cb`) with
  a category-coloured uppercase title over a bottom-border divider (no left-accent bar, no dot).
- `Bar` + `Breakdown` - horizontal contribution bars for the "why this score" panel.
- `Seg` - shared segmented control (pill group on a `th.seg` track; active option gets `th.acc`). Used
  for theme + engine in Settings and the compare target in the Import modal.
- Line-icon components (`Ic*`, built on a shared `Svg` wrapper): camera, compare, save, chart, users,
  history, settings, sun, moon, trophy, trash, load (log-in/enter glyph), reset (rotate-ccw). Used on
  the top action row, row actions (History load/delete, Players delete), name-field reset, and modal
  headers.

`App` holds all state and renders the page chrome (header, tip, panels, sticky results bar, mode tabs,
OCR modal) as inline JSX - inline elements keep stable identity across renders, so inputs there (name
fields, modal fields) do not lose focus.

## Pure helpers (module scope)

`tierOf`, `calcImpact`, `contribsOf`, `deltaContribs`, `sortKeys`, `fmt`, `colFor`/`dcolFor`,
`load`/`save` (localStorage), plus the OCR helpers (`lev`, `matchName`, `pairFuzzy`, `parseOcr`,
`upscale`, `geminiExtract`, `tesseractExtract`). Attribute groups (`OFF`, `DEFN`, `GKL`, `PH`, `ME`,
`TE`, `HI`) and the default player `DEF` are module constants. The impact math (`imp`, `calcImpact`)
is unchanged from the original.

## Page layout (top to bottom)

1. **Sticky bar** (`sticky top-0 z-30`, blurred, theme-aware) - simplified to a single row:
   - Logo + title, and the icon-action row (Import, Compare, Save, Breakdown, Players, History, Settings).
2. **Hero scoreboard** - a card below the sticky bar.
   - Single mode: editable player name (reset icon on the right) + Points / GF / GA metrics.
   - Compare mode: a fixed `md:grid-cols-3` — P1 (reset icon on the left + name + points) | centre
     **Difference** block (Pts/GF/GA deltas + verdict) | P2 (name + reset icon on the right + points),
     split by vertical dividers (`border-x`), no nested box. Because the columns are a fixed 1/3 grid and
     every number uses `tabular-nums` with fixed-width delta cells (`dStat`, `w-12`), the layout does not
     shift as values change. Resets sit on the outer edge next to each player, not by the centre.
3. **First-run tip** - dismissible banner pointing at Import; hidden once dismissed (`fm_seen_tip`).
4. **Mode tabs** - All / Offensive / Defensive / Goalkeeper + active-attribute count; the **Only
   differences** filter (`onlyDiff`) sits at the right of this row (compare only).
5. **Breakdown** panel (toggle) - per-attribute Points contribution bars; compare shows the P1−P2 delta.
6. **Saved players** panel (toggle) - roster in `localStorage`. Save/Update slot (the button reads
   **Update** when a saved player already matches the current name, else **Save**), search box (shown once
   >3 players), per-player mode badge (GK/Off/Def/All, colour-coded), load into P1/P2, delete.
7. **History** panel (toggle) - saved snapshots (load/delete) with player names.
8. **Attribute grid** - `Cat` cards in a **stable balanced masonry**: `ncol` is derived from a tracked
   `vw` (window width) at the same breakpoints (single: 3 @≥1024, 2 @≥640, else 1; compare: 2 @≥1280,
   else 1), then categories are greedily packed into `ncol` flex columns by attribute count (`catCols`).
   This packs short categories under tall ones (no big bottom gap, e.g. Hidden under Physical) while
   staying put on value edits (assignment depends only on the visible attribute counts, not values).
   Column width is always `1/ncol` regardless of how many categories a mode shows, so widths stay
   consistent when switching modes. Compare uses a centre-aligned "tug-of-war" row layout. Goalkeeper is
   a single card rendered in the same `1/ncol`-width grid (`gkGrid`) so its width matches the others.
9. **Footer** - one centred line (divider above) crediting the weight source.
10. **Settings modal** (gear) - theme, OCR engine, Gemini API key, reset-all, Help + Tiers reference.
11. **Import modal** - see `docs/screenshot-autofill.md`.

The old compare action row (Copy P1→P2 / Swap / per-slot Reset) was removed as confusing: reset moved
to a per-player icon on the scoreboard, and Copy/Swap were dropped. The "Biggest differences" strip was
also removed earlier; the centre Δ block + Breakdown panel cover that need.

## localStorage keys

- `fm_history` - saved snapshots (array, capped at 20).
- `fm_roster` - saved players `{name, isGk, mode, data, ts}` (array, capped at 50).
- `fm_gemini_key` - the user's Gemini API key (bring-your-own).
- `fm_ocr_engine` - last chosen OCR engine (`gemini` | `tesseract`).
- `fm_seen_tip` - `'1'` once the onboarding tip is dismissed.
- `fm_prefs` - persisted UI preferences `{dm, m, c, onlyDiff, showBreak, showRoster, showHs}`
  (theme, last mode, compare on/off, only-differences filter, and which panels were open). Read once
  via a lazy `useState` for initial values, then written by an effect on any change. Theme defaults to
  dark when unset (`dm !== false`).

## Undo toast

Destructive actions (reset-all in Settings, delete a history snapshot, delete a saved player) do **not**
use confirm modals. Instead they run immediately and pop a lightweight bottom-centre toast
(`{msg, undo}`, `z-[60]`, auto-dismiss after 6s) with an **Undo** button that restores the prior state
(deletes re-insert at the original index; reset-all restores the full P1/P2 snapshot). Saving a player
also toasts (Undo removes/reverts the entry). Managed by `showToast`/`doUndo` + a `toastRef` timer.

## Theme

`th` is a flat object (dark or light) mapping semantic names (bg/card/border/text/input/button/`acc`)
to Tailwind classes; every component takes `th` so styling stays centralized. Design rules:

- **One action accent:** amber-500 (`th.acc` = active/toggled button fill; also focus rings, links,
  the logo badge, and section headers). No gradients.
- **Intentional colour elsewhere:**
  - Top action row uses small **line icons** (`Ic*`) with labels that hide below `md`.
  - **Tier-coloured attribute letters** (`TIER_TEXT`: S rose, A orange, B amber, C emerald, D sky).
  - **Per-category title colour** (`CAT`: Physical emerald, Mental sky, Technical amber, Hidden violet,
    Goalkeeper rose) — applied only to each `Cat` card's uppercase title (no left-accent bar, no dot).
  - Green (`emerald-500`) / red (`rose-500`) encode meaning: Points/GF/GA metrics, compare deltas,
    breakdown bars, diff chips (`POS`/`NEG`/`NEU`/`ACC` + `colFor`/`dcolFor`).

## Actions & Settings

All actions live in a **single top row** inside the sticky bar (logo left; icon+label buttons right:
Import, Compare, Save, Breakdown, Players, History, Settings) — never a second button row. The
**Settings modal** (gear) consolidates the low-frequency controls as harmonised labelled rows: theme
and OCR engine as `Seg` segmented controls, Gemini API key input, and a reset-all action. Below a
divider, Help + Attribute-tiers are **static reference sections** (no dropdowns/`<details>`). The
Gemini key is entered here; the Import modal only shows a small "Change in Settings" line.

## Density

The attribute cards use a **JS-balanced flex masonry** (greedy shortest-column packing over a `vw`-derived
`ncol`) rather than CSS `columns-*`. CSS multi-column reflowed cards across columns whenever content
height changed, so cards jumped left/right; a plain row-major `grid` fixed the jump but left a big gap
under short columns (e.g. Hidden far below Physical in All mode). The balanced-column approach fixes both:
no reflow on value edits (assignment depends only on visible attribute counts) and no bottom gap (short
categories pack under tall ones). Toggle panels (Breakdown, Players, History) are closed by default.
