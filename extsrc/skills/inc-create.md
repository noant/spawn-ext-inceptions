---
name: inc-create
description: Entry point of the inceptions methodology. Request classification (question/initiative), language detection, clarification, creation of inception and attempt (motivation.md, overview.md, res/).
---

# inc-create

Entry point of the inceptions methodology. Role: `inc-engineer` (main chat).

Full methodology: `inceptions/main.md`. Follow it.

## What it does

Classifies the user request, runs the synthesis check, and routes it: answers a question (Stage 1), runs research and creates an inception/attempt (Stage 2), or continues an existing inception.

## Order

1. Read `inceptions/main.md` fully.
2. Synthesis check (Stage 0.0, `inc-rule-synthesis`): if the request involves another methodology/skill — offer synchronous work (see main.md); if none detected — skip.
3. Determine the user's language (`inc-rule-language`): unambiguous — work in it; ambiguous — offer a set via `inc-rule-ask`.
4. Classify the request (Stage 0): **[A] Question** (asks) or **[B] Initiative/task** (explicit "do" / research).
5. If unclear — ask clarification (`inc-rule-ask`).
6. If the request relates to an existing inception in `inceptions/` — propose to continue (use `inc-continue` to determine where).
7. **[A] Question** → Stage 1 (Question, "light" mode — no inception folder is created). Follow the full flow 1.1-1.6 in main.md: clarify → decompose the question → organize researchers → distribute sub-questions → synthesize the answer → if an inception is clear, propose Stage 2.
8. **[B] Initiative/task** → Stage 2 (Research). Follow the full flow in main.md: clarify the initiative (2.1) → create base docs first (2.2: if first attempt of first inception — create `inceptions/{N}-{inception-slug}/` and `motivation.md`; create the attempt folder `try-{N}-New-{description}/`, `overview.md`, `res/`) → ask research depth (2.3, default `[medium]`) → research loop up to 3 waves (2.4: decompose → launch researchers → collect into `overview.md` → review via `inc-reviewer` → decide) → continuation proposal (2.5) → user research review (2.6: `[Proceed]`/`[Refine]`/`[Stop]`).

Steps 7 and 8 are branches, not a sequence — pick the one matching the request type.

## Launching a researcher subagent

When research is needed (type A answer; type B research loop), launch a subagent via the task tool. Role choice per the role-choice guidance in main.md: `inc-explorer` for decomposition into independent lines, `inc-researcher` for a narrow single-line question (default for type A), inline only for one-off simple ops. Build the prompt by the "Direction line template" from main.md; put the resolved Ambient block at the start.

## Templates

- Artifacts: `inceptions/templates/motivation.md`, `overview.md`, `technical-task.md`, `result.md`, `rule.md`.
- Subagent responses: `inceptions/templates/executor-report.md`, `researcher-report.md`, `reviewer-report.md`, `final-report.md`.
