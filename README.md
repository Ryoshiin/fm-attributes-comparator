# FM Attributes Comparator

A browser tool for Football Manager. It turns a player's attributes into a projected match impact
and lets you compare two players side by side.

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
- Compare two players and see the delta.
- Save comparisons to a local history (stored in the browser via `localStorage`).
- Attribute tier list and an in-app help panel.

### Planned

- **Screenshot autofill** - upload an FM player profile screenshot; in-browser OCR (Tesseract.js)
  reads the attribute values and fills the inputs automatically. See
  [`docs/screenshot-autofill.md`](docs/screenshot-autofill.md).
- Migrate the remaining French UI text to English.

## Run it

No build step. Either:

- Open `src/index.html` directly in a browser, or
- Serve it (needed later so the browser can load screenshot files reliably):

```bash
cd src
python -m http.server 8000
# then open http://localhost:8000
```

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
