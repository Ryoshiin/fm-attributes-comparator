---
name: route
description: Assess whether the current work fits the lightweight everyday lane or should escalate to the full BMad Method, and if escalating, output a concrete handoff (artifact inventory, install steps, source-to-BMad mapping, post-handoff ownership). Use when unsure how much process a project or task needs.
---

# route

Decide the right amount of process and, if the work is too big, hand it off to BMad cleanly. The
everyday lane (this template) is for small, solo, clear-scope work. Bigger or higher-stakes work
belongs in the full BMad Method, which is scale-adaptive and far stronger at scale.

## How to assess

Ask a few questions only if needed, then classify against the rubric. Lean to the everyday lane
when genuinely borderline - but ANY single hard trigger forces escalation regardless of size.

### Hard escalation triggers (any one -> BMad)

- Security-sensitive, auth, or payments code; handles regulated data or PII; compliance scope.
- Multiple contributors who need shared specs and sequencing.
- A durable, non-trivial data model, or a schema migration.
- A public API others depend on, or multi-tenant.
- Real deployment/operational risk (downtime or data loss) on change.
- A large migration or refactor spanning many modules.

### Size signals (several -> BMad; few -> everyday)

Everyday lane (Level 0-1):

- One or few contributors; you are the main user/owner.
- Scope fits in your head; few moving parts.
- Roughly < 10 distinct work units; days, not weeks.

Escalate (Level 2+):

- New product/platform with many features (~10+ distinct work units).
- Multiple domains, services, or integrations that must stay coherent.
- Multi-week scope, or architecture that is not yet clear.

If borderline with no hard trigger, recommend starting in the everyday lane and re-running `route`
if it grows.

## Output

State the verdict in one line: `Everyday lane` or `Escalate to BMad`, with a one-sentence reason
naming the trigger or signals that drove it.

### If everyday lane

Recommend the next everyday step (`define-project`, `brainstorm`, or just start working).

### If escalate to BMad

Output the full handoff. Do not run commands yourself - give the user the steps.

1. Artifact inventory - list the context that already exists to carry over, and flag each as
   current, stale, or missing:
   - `README.md`, the `AGENTS.md` Project section, every file in `docs/`, and `tasks.md`.
   - Call out gaps BMad will need the user to fill (e.g. no architecture, thin requirements).
2. Install: `npx bmad-method install` in the project root; choose Cursor as the IDE; install the
   `bmm` module (core). Requires Node / npx. BMad adds its own files; it should not edit your
   `AGENTS.md`, `README.md`, or `docs/` - confirm nothing of yours is overwritten. (For a brand-new
   project known to be big up front, `np <name> --bmad` scaffolds and runs this install in one step.)
3. Source -> BMad mapping (seed BMad's planning instead of starting blank):
   - `README.md` + `AGENTS.md` Project Overview -> BMad product brief.
   - `docs/` topical docs -> BMad PRD / requirements.
   - Every open `[ ]` and `[~]` item in `tasks.md` -> BMad epics / stories; paused-work notes ->
     BMad open issues / next actions. Convert them explicitly; do not lose in-flight context.
   - `AGENTS.md` Rules / Boundaries / Conventions -> keep as-is; BMad layers workflows on top.
4. Transfer tracking ownership (exactly one system of record):
   - Move `tasks.md` to `docs/legacy-tasks.md` (preserves the dated history).
   - Replace root `tasks.md` with a one-line pointer ("Tracking moved to BMad; history in
     `docs/legacy-tasks.md`."), or delete it if you prefer no stub.
   - From here BMad owns all new planning and tracking; stop adding backlog to the everyday files.
   - Keep `AGENTS.md` as the shared role/rules backbone; both lanes read it.
5. Run exactly ONE heavy lane - never BMad alongside another spec framework.
6. Ask `bmad-help` in the project for "what's next".

## Rules

- A single hard trigger outranks all size signals.
- Do not install BMad or run shell commands; output the steps for the user to run.
- Do not start planning or implementation; this skill only assesses and hands off.
