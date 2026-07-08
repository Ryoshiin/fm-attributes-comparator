# AGENTS.md

Canonical instructions for AI agents in this project. This file is the source of truth.
`CLAUDE.md` points here; skills live in `.agents/skills/`.

Two regions:

- The FRAMEWORK block below is template-managed. `np update` overwrites it, so do not hand-edit it
  expecting changes to survive an update.
- The `## Project` section at the bottom is project-owned. `np update` never touches it.

Keep this file lean (target <= 180 lines). Put occasional, task-specific procedures in
`.agents/skills/`, not here.

<!-- BEGIN FRAMEWORK (template-managed; np update overwrites this block; keep project content OUTSIDE it) -->

## Role

Maximize the user's autonomy and correctness over engagement. Be a precise collaborator, not a
cheerleader.

## Rules (resolve conflicts by order)

1. Blunt, directive phrasing. No emojis, filler, hype, transitions, or flattery.
2. Output only what the task requires. No preamble or closing padding.
3. Ask a clarifying question only when ambiguity blocks correctness, never for engagement.
4. Challenge wrong assumptions; state errors, risks, and limitations directly.
5. Verify factual or time-sensitive claims against primary sources; if unverifiable, say so.
6. Prefer the smallest correct change. Do not refactor unrelated code.
7. Do not invent project facts. Read the docs first (see Project section + `docs/`).
8. Project-specific rules belong in the `## Project` section (Conventions / Non-inferable rules).
   They may override framework process or style defaults for this project, but never the Boundaries
   or the Definition of Done.

## How this project works (everyday lane)

A lightweight, living-documentation workflow:

- `AGENTS.md` (this file) - how to work here + project constraints.
- `README.md` - human-facing overview.
- `tasks.md` - backlog by topic + a dated "Done / notes" log = the change history.
- `docs/` - topical, self-contained docs = the current-state source of truth.
- `local/` - git-ignored private scratch; create `logs/`, `secrets/`, etc. on demand (never read `local/secrets/`, never commit anything under `local/`).

No epics, stories, or per-story reviews here. If a project outgrows this lane, run the `route`
skill; it hands off to the full BMad Method (`npx bmad-method install`).

### Skills (`.agents/skills/`)

Portable Agent Skills, read natively by Cursor, Codex, and Gemini. For Claude Code, mirror them to
`.claude/skills/` via `np <name> --claude`.

- `define-project` - set up or refine this project (overview, stack, structure, initial tasks).
- `brainstorm` - structured ideation to pin down scope / boundaries before committing.
- `route` - assess size; continue here or escalate to BMad.

## Living-docs convention (the core discipline)

A change is "meaningful" when it changes behavior, structure, commands, config, dependencies, or
decisions, or introduces a gotcha. Trivial edits (typos, formatting, comments) need no doc update.

For a meaningful change, in the SAME change:

- `tasks.md` - set the item to `[x]` and add a dated "Done / notes" entry (what changed, why, any
  gotcha). See `tasks.md` for the format.
- `docs/` - update the topical doc it affects. Create `docs/<topic>.md` only if none fits; never
  leave a stale doc - fix or delete it in the same change.
- `## Project` (below) - update ONLY if stack, structure, commands, or constraints changed.

Pausing mid-task: set the item to `[~]` in `tasks.md` with a paused-work note (Context / Current
state / Next step / Validation / Risks - see `tasks.md`). That note is how the next session
resumes safely. Continuity comes from these files, not chat history.

## Code quality (code archetypes)

Applies to code projects (`code-app`, `library-cli`); docs/infra projects organize by topic instead.

Principles - apply pragmatically to cut real complexity, never as ceremony:

- KISS - choose the simplest solution that works; avoid clever indirection.
- YAGNI - build only what the current task needs; no speculative features, abstractions, or config.
- SOLID - single-responsibility, focused units that depend on clear interfaces; apply where it
  removes real complexity, not by default.

Structure:

- Organize by responsibility or feature; keep each module focused and files small.
- Put shared/reusable code in one obvious place (e.g. `lib/` or `utils/`); do not scatter duplicates.
- Colocate tests with the code they cover, or mirror the source tree under `tests/`.
- Put new code where similar code already lives; follow the layout recorded in `## Project` (Structure).
- Do not restructure or move modules without a reason recorded in `tasks.md`.

Comments - explain WHY, not WHAT; prefer self-explanatory code:

- Comment intent, complex logic, edge cases, assumptions, and non-obvious context only.
- Do NOT narrate code a reader can already see, restate the obvious, or comment every function by rule.
- A comment should add value and never be longer than the code it describes.
- Make code you are touching self-explanatory rather than explaining unclear code in a comment.
- Plain sentences. No decorative banners, ASCII dividers (`-----`), or TITLE headers.

## Boundaries / Do-Not

- Never commit anything under `local/`. Never read or edit `local/secrets/`.
- Never commit secrets, `.env` files, credentials, tokens, keys, or generated build output.
- Require explicit confirmation before: deleting files/folders, running destructive commands,
  changing auth/payments/security code, DB schema or migrations, major dependency upgrades, or
  editing lockfiles unless a dependency change requires it.
- Do not create commits or stage files unless the user explicitly asks.
- Do not add dependencies unless the benefit is clear; document why when you do.

## Definition of Done (self-verify before declaring done)

- [ ] The change satisfies the request and nothing out of scope was changed.
- [ ] Validation ran (tests / lint / build / manual check) or you stated why it could not.
- [ ] `tasks.md` updated: item set to `[x]` + dated "Done / notes" entry added.
- [ ] Relevant `docs/` updated in the same change; Project section updated if it changed.
- [ ] No secrets or `local/` content were committed.

## Template protection

If the working folder is the template itself (`ryo-workflow`), do NOT run `define-project` or
treat it as a real product. Make template-maintenance edits only when the user is clearly editing
the template.

<!-- END FRAMEWORK -->

## Project

- Overview: A browser tool for Football Manager players. It scores a player's attributes into a
  projected match impact (Points, Goals For, Goals Against) using FM Arena testing weights, compares
  two players side by side, and keeps a local history. Screenshot autofill is shipped: upload/paste/
  drag an FM player profile screenshot and a cloud/offline reader fills the attribute inputs.
- Type / archetype: code-app (single-file static web app, no build step).
- Stack / platform: Single self-contained `src/index.html`. React 18 + ReactDOM + Babel Standalone
  (JSX transpiled in-browser) and Tailwind, all via CDN. Screenshot autofill uses Google Gemini Flash
  (primary, bring-your-own free API key) with Tesseract.js (CDN) as an offline fallback. No package
  manager, bundler, or server. State in `localStorage` (`fm_history`, `fm_roster`, `fm_gemini_key`,
  `fm_ocr_engine`, `fm_seen_tip`).
- Structure:
  - `src/index.html` - the entire app in a `<script type="text/babel">` block. Data + pure helpers +
    presentational components live at **module scope** (top of the script); `App` holds all state and
    the page chrome. Key data: `attr` (key -> label), `imp` (key -> weights `{p, gf, ga}`), the mode
    lists (`OFF`/`DEFN`/`GKL`) and category lists (`PH`/`ME`/`TE`/`HI`). Components with attribute
    inputs (`Num`, `AttrRowSingle`, `AttrRowCompare`, `Cat`, `Breakdown`) MUST stay at module scope
    and take `th`/handlers as props - defining them inside `App` remounts inputs and drops focus. Add
    attribute data by extending the objects; add UI as module-scope components. See `docs/ui.md`.
  - `docs/` - topical living docs (`docs/impact-model.md`, `docs/screenshot-autofill.md`, `docs/ui.md`).
  - `tests/` - present but unused (no test runner for this single-file app); verify by opening the
    HTML in a browser.
  - `scripts/`, `local/` - helper scripts and git-ignored scratch.
- Key commands: No build. Open `src/index.html` directly in a browser. Fastest local serve:
  `python -m http.server 8000` from `src/` (or any static server), then load `localhost:8000`.
- Conventions:
  - UI text is English. Date/time uses the `en-GB` locale; attribute sorting uses the `en` locale.
  - Design: clean/dense, amber (`th.acc`) as the single action accent (no gradients). Colour is used
    with intent — line-icons on the top action row, tier-coloured attribute letters (`TIER_TEXT`), and
    per-category accent colours (`CAT`: emerald/sky/amber/violet/rose) for grouping; green/red encode
    meaning (metrics, deltas, breakdown bars). Actions live in ONE top row + a Settings modal (API key,
    OCR engine, theme, reset, help, tiers); don't spill buttons into a second row. Keep the default
    view fitting without scrolling. See `docs/ui.md` before restyling.
  - Attributes are keyed by camelCase English FM names (e.g. `firstTouch`, `workRate`, `gkReflexes`).
    Screenshot OCR must map on-screen FM attribute names to these keys.
  - Attribute scale is 1-20; 8 is the neutral reference (impact 0). Impact = `weight * (value-8)/12`.
  - Keep the app dependency-free beyond CDN scripts; do not introduce a build step or npm.
- Non-inferable rules:
  - Impact weights in `imp` come from FM Arena FM24 attribute testing (38 matches); they are
    empirical, not derived. Do not "correct" them without a cited source.
  - Short identifiers are common (`p1`, `hs`, `dm`, `th`, `c`). The UI/UX overhaul loosened the
    minified style in the parts it touched (module-scope components, readable JSX); match the local
    style of the code you edit and do not reformat the whole file.
  - Babel Standalone transpiles JSX at runtime, so a syntax error shows only in the browser console,
    not at edit time - check the console after changes.
