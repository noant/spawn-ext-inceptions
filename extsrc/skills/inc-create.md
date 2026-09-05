---
name: inc-create
description: Entry point of the inceptions methodology. Request classification (question/initiative), language detection, clarification, creation of inception and attempt (motivation.md, overview.md, res/).
---

# inc-create

Entry point of the inceptions methodology. Role: `inc-engineer` (main chat).

Full methodology: `inceptions/main.md`. Follow it.

## What it does

Classifies the user request and, if necessary, creates an inception and an attempt.

## Roles that can be called

- `inc-explorer` — coordinating researcher (decomposes research into lines).
- `inc-researcher` — executing researcher (narrow single-line research).
- `inc-executor` — executor (performs work by a spec).
- `inc-reviewer` — reviewer (reviews research/tasks/work).

## Order

1. Read `inceptions/main.md` fully.
2. Determine the user's language (`inc-rule-language`):
   - If unambiguous — work in it, do not offer a choice.
   - If ambiguous — offer a set of languages via `inc-rule-ask`, based on context.
3. Classify the request (Stage 0):
   - **[A] Question** — the user asks, not requests to "do".
   - **[B] Initiative / task** — an explicit "do" request or a request to conduct research.
4. If something is unclear — ask clarification (`inc-rule-ask`).
5. If the request relates to an existing inception in `inceptions/` — propose to continue working with it (use `inc-continue` to determine where to continue).
6. For a type B request — move to Stage 2 (Research):
   - clarify the initiative, ask for motivation;
   - **create the base documentation first** (Stage 2.2, mandatory before any research):
     - if first attempt of first inception — create `inceptions/{N}-{inception-slug}/` and `motivation.md`;
     - create the attempt folder `try-{N}-New-{description}/`, `overview.md`, `res/`;
     - **checklist:** `motivation.md` (first attempt), `try-{N}-New-{description}/`, `overview.md`, `res/` — all must exist before research starts;
   - ask the research depth (Stage 2.3): `[inline]` / `[medium]` (single explorer) / `[high]` (several directed explorers); default `[medium]`;
   - **launch researcher subagents** per the chosen depth (Stage 2.4).
7. For a type A request — move to Stage 1 (Question):
   - **launch a researcher subagent** (`inc-explorer` or `inc-researcher`) to answer the question (Stage 1.3).

## Launching a researcher subagent (mandatory step)

When the request reaches a point that needs research (type A — answering the question; type B — the research loop), launch a subagent via the task tool:

- Use `inc-explorer` when the research decomposes into several independent lines (parallel/sequential sub-lines).
- Use `inc-researcher` when the research is narrow and single-line.
- Use inline (no subagent) only for one-off simple operations where launching a subagent is excessive.

Build the subagent prompt by the "Direction line template" from `inceptions/main.md` (role, agent slug, question, hypotheses, sources, constraints, research file). Put the resolved Ambient block at the start of the prompt.

## Researcher selection (preference)

1. `inc-explorer` — preferred choice for research.
2. `inc-researcher` — for small, narrow research.
3. inline (no subagent) — for one-off simple operations.

## Templates

- Artifacts: `inceptions/templates/motivation.md`, `overview.md`.
- Subagent responses: `inceptions/templates/agent-responses.md`.
