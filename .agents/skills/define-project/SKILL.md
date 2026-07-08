---
name: define-project
description: Set up or refine the current project - fill the AGENTS.md Project section, README, and an initial tasks.md, and scaffold folders for the chosen archetype. Use at project start or when scope/stack/structure changes.
---

# define-project

Turn a project idea into a working everyday-lane setup: a filled-in `AGENTS.md` Project section, a
short `README.md`, an initial `tasks.md`, and the right folders for the project type.

This is the everyday-lane replacement for heavyweight planning. Keep it low-friction. It does NOT
create epics, stories, or per-story review files. For anything bigger, use the `route` skill.

## When to use

- Starting a new project from this template.
- The project's purpose, stack, structure, or constraints changed and the docs are now stale.

## Prerequisites

- Do NOT run this in the template itself (`ryo-workflow`). See AGENTS.md "Template protection".
- If BMad is already installed in this project (e.g. a `bmad`/`.bmad*` folder or BMad config
  exists), STOP: BMad owns planning. Do not run everyday-lane setup on top of it.

## Process

1. If the idea is fuzzy or the boundaries are unclear, offer to run `brainstorm` first. If a
   brainstorm doc exists in `docs/`, read it.
2. Ask only for what you cannot infer. Keep the first round to <= 6 questions, grouped:
   - What is it and who is it for?
   - What does success / "done enough" look like for v1?
   - Archetype (see list) or let me infer it.
   - Stack / platform, if known. Out of scope, if known.
   - Any non-inferable rules or hard constraints (gotchas, environment, deadlines).
3. Choose an archetype and scaffold only the folders that archetype needs (see Archetypes).
4. Write the outputs (see Outputs). Keep everything short and current.
5. Self-assess scale: before stopping, judge the project against the `route` skill's rubric (hard
   escalation triggers + size signals). Read `route`; do NOT duplicate its rubric here. Decide
   yourself whether this belongs in the everyday lane or BMad.
6. Stop point: this skill ends once the outputs exist and are accurate. Do not start building.
   Give a one-line routing verdict and the matching next action:
   - Everyday lane -> start working (or `brainstorm` if scope is still fuzzy).
   - Escalate to BMad -> say so plainly, name the trigger/signals, and run `route` for the full
     handoff. (For a fresh project the user can instead scaffold + install in one step with
     `np <name> --bmad`.) Do not start everyday-lane implementation.

## Archetypes

Pick the closest; scaffold only what it needs. Do not create folders the project will not use.

- `code-app` - app / service. Add `src/`, `tests/`. Record build/test/run commands.
- `library-cli` - reusable lib or CLI. Add `src/`, `tests/`. Record build/test/publish commands.
- `infra-ops` - infrastructure / homelab / ops. Add `scripts/` and topical `docs/`. Record ops
  commands and deployment model.
- `docs-kb` - documentation or knowledge base. Use `docs/` + `tasks.md`. No `src/`.
- `automation-research` - scripts, experiments, notes. Add `scripts/`; keep notes in `docs/`.

If unclear, ask the user to pick one. Do not invent a non-code structure for a code project or
vice versa.

## Outputs

- `AGENTS.md` Project section - fill every field (Overview, Type/archetype, Stack/platform,
  Structure, Key commands, Conventions, Non-inferable rules). Edit ONLY the `## Project` section
  (project-owned); never edit inside the `FRAMEWORK` block. Keep the whole file <= 180 lines.
  For code archetypes, make `Structure` capture code organization - where each kind of code goes,
  module boundaries, and naming - not just a folder list. The FRAMEWORK "Code structure" rules
  defer to it, so it is where this project's specifics live.
- `README.md` - human-facing: what it is, how to run / use it, where things live.
- `tasks.md` - seed the Backlog with concrete v1 tasks grouped by topic. Leave the "Done / notes"
  log empty (keep the dated-entry format).
- Folders - create only the archetype's folders. You MAY create empty code folders (`src/`,
  `tests/`) with a `.gitkeep`, but do NOT write source code or implement features here.
- `docs/` - add a stub doc only when a topic clearly needs its own page; do not pre-create empty docs.

## Rules

- Setup only: do not write source code or implement features.
- Do not over-specify. Mark unknowns under "Open questions" in the README rather than inventing.
- Do not duplicate content across files: AGENTS.md = how to work + constraints; README = human
  overview; tasks.md = backlog + history.
- Update existing content instead of blindly overwriting it.

## Completion

- AGENTS.md `## Project` section is complete and accurate; the FRAMEWORK block is untouched.
- README and tasks.md exist and reflect the real project.
- Only the archetype's folders were created.
- No code written, no epics/stories created.
- A one-line routing verdict (everyday lane vs escalate to BMad) was given.
