# FM Attributes Comparator

A browser tool for Football Manager. It turns a player's attributes into a projected match impact
and lets you compare two players side by side.

**Live:** https://ryoshiin.github.io/fm-attributes-comparator/

For each player it computes three numbers from the attribute values:

- **Points** - overall projected impact on results.
- **Goals For (GF)** - projected attacking contribution.
- **Goals Against (GA)** - projected defensive effect (lower is better).

The scoring uses empirical attribute weights from
[FM Arena's FM24 attribute testing](https://fm-arena.com/thread/14009-attribute-testing-football-manager-24/)
(38 matches). See [`docs/impact-model.md`](docs/impact-model.md) for the exact model.

## Features

- Enter attributes (1-20) and see projected Points / GF / GA live.
- Modes: All, Offensive, Defensive, Goalkeeper - each scores a relevant subset of attributes.
- Compare two players with a Δ scoreboard and a per-attribute breakdown.
- **Screenshot autofill** - upload/paste/drag an FM player profile screenshot and auto-fill the
  attributes, via Gemini (cloud, bring-your-own free key) or Tesseract (offline). See
  [`docs/screenshot-autofill.md`](docs/screenshot-autofill.md).
- Save players (roster) and comparison history in the browser (`localStorage`).
- Attribute tier list and in-app help (in the Settings panel).

## Run it

No build step. Either:

- Use the hosted version: https://ryoshiin.github.io/fm-attributes-comparator/, or
- Open `src/index.html` directly in a browser, or
- Serve it locally (so the browser can load screenshot files reliably):

```bash
cd src
python -m http.server 8000
# then open http://localhost:8000
```

## Deploy

Pushing to `master` auto-publishes `src/` to GitHub Pages via
[`.github/workflows/deploy-pages.yml`](.github/workflows/deploy-pages.yml).

## How it works

The whole app is one self-contained file, `src/index.html`: React + Tailwind loaded from CDNs, with
the app written as a single `App` component that Babel transpiles in the browser. Attribute
definitions and impact weights are plain objects at the top of the script. There is no server; the
comparison history lives in `localStorage`.

## Where things live

- `src/index.html` - the entire application.
- `docs/` - the impact model and the screenshot-autofill design.
- `tasks.md` - backlog and dated change log.
- `AGENTS.md` - instructions and constraints for AI agents working in this repo.

## Open questions

- OCR accuracy on FM screenshots and how much manual correction the autofill UI needs.
- Whether GK attributes appear reliably on the same screenshot layout as outfield players.
