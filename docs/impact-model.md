# Impact model

How the app turns attribute values into projected match impact.

## Inputs

- Each attribute has a value on the FM scale of **1-20**.
- **8 is the neutral reference**: an attribute at 8 contributes 0 impact.

## Formula

For each attribute in the active mode's subset, with value `v` and weights `{p, gf, ga}`:

```
multiplier = (v - 8) / 12
Points        += p  * multiplier
Goals For     += gf * multiplier
Goals Against += ga * multiplier
```

Totals are summed across the active attributes. `(v - 8) / 12` maps value 8 -> 0 and value 20 -> 1
(and 1 -> roughly -0.58), so weights express the impact of a "maxed" attribute relative to neutral.

The weights live in the `imp` object at the top of the script in `src/index.html`. Goalkeeper
attributes are keyed with a `gk` prefix (e.g. `gkReflexes`) and weighted separately.

## Modes

Modes select which attributes are scored:

- **All** - every attribute.
- **Offensive** - the `off` list (attacking-relevant outfield attributes).
- **Defensive** - the `def` list (defending-relevant outfield attributes).
- **Goalkeeper** - the `gk` list only.

The exact per-mode lists are defined as `off`, `def`, and `gk` arrays in `src/index.html`.

## Source

Weights come from FM Arena's FM24 attribute testing over 38 matches:
<https://fm-arena.com/thread/14009-attribute-testing-football-manager-24/>. They are empirical
measurements, not derived values - do not change them without a cited source.
