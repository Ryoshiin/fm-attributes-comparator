---
name: brainstorm
description: Structured ideation to explore an idea and pin down scope and boundaries before committing. Offers a technique menu and two modes, then writes a captured session doc. Use when an idea is fuzzy or you need options before defining or building.
---

# brainstorm

A low-friction, technique-driven brainstorming session. Distilled from established facilitation
practice: it guides idea generation instead of just chatting, then captures the result so it feeds
`define-project` or the work itself.

## When to use

- The idea or its boundaries are unclear.
- You need divergent options before narrowing to a decision.

## Modes

Ask which mode, default to Interactive:

- Interactive - you facilitate: ask one focused prompt at a time, build on answers, reflect back.
- Quick - you generate: produce a structured set of options/ideas fast, then let the user react.

## Technique menu

Offer a short menu and use 1-3 that fit. Do not run all of them.

1. What if - provocative "what if X" scenarios to stretch the space.
2. First principles - strip to fundamentals, rebuild from there.
3. SCAMPER - Substitute, Combine, Adapt, Modify, Put to other use, Eliminate, Reverse.
4. Role-storming - view the problem as different stakeholders / users.
5. Five whys - chase a problem to its root cause.
6. Assumption reversal - list assumptions, deliberately invert them.
7. Analogies - solve by comparison to an unrelated domain that solved something similar.
8. Constraints - impose a hard limit (no budget, one day, one file) to force creativity.

## Process

1. Confirm the topic / question in one line.
2. Pick mode and 1-3 techniques.
3. Run the session. Diverge first (generate), then converge (cluster, rank, pick).
4. End with: top ideas, key boundaries / scope decisions, and open questions.
5. Write the captured doc (see Output). Recommend `define-project` next if this was pre-setup.

## Output

Default to durable project context:

- `docs/brainstorm-<slug>.md` with: Topic, Techniques used, Ideas (grouped), Decisions /
  boundaries, Open questions, Next step.
- Use `local/brainstorm-<slug>.md` (git-ignored) ONLY for a private / throwaway session you do not
  want in project history.

## Rules

- Generate before judging; do not collapse to one answer too early.
- Keep the captured doc concise; it is a working artifact, not a transcript.
- Do not start implementing. This skill explores and captures only.
