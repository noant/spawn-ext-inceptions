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
   - if first attempt of first inception — create `inceptions/{N}-{inception-slug}/` and `motivation.md`;
   - create the attempt folder `try-{N}-New-{description}/`, `overview.md`, `res/`.
7. For a type A request — move to Stage 1 (Question).

## Researcher selection (preference)

1. `inc-explorer` — preferred choice for research.
2. `inc-researcher` — for small, narrow research.
3. inline (no subagent) — for one-off simple operations.

## Templates

- Artifacts: `inceptions/templates/motivation.md`, `overview.md`.
- Subagent responses: `inceptions/templates/agent-responses.md`.
