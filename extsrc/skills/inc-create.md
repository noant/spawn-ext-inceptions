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
2. Synthesis check (Stage 0.0) — see `inc-rule-synthesis-check` in main.md.
3. Determine the user's language (`inc-rule-language`): unambiguous — work in it; ambiguous — offer a set via `inc-rule-ask`.
4. Classify the request (Stage 0): **[A] Question** (asks) or **[B] Initiative/task** (explicit "do" / research).
5. If unclear — ask clarification (`inc-rule-ask`).
6. If the request relates to an existing inception in `inceptions/` — propose to continue (use `inc-continue` to determine where).
7. **[A] Question** → Stage 1 (Question, "light" mode — no inception folder is created). Follow the full flow 1.1-1.6 in main.md.
8. **[B] Initiative/task** → Stage 2 (Research). Follow the full flow 2.1-2.6 in main.md (base docs first, research depth, research loop up to 3 waves, continuation proposal, user research review).

Steps 7 and 8 are branches, not a sequence — pick the one matching the request type.

## Launching a researcher subagent

When research is needed (type A answer; type B research loop), launch a subagent via the task tool. Role choice per `inc-rule-role-choice` in main.md: `inc-explorer` for decomposition into independent lines, `inc-researcher` for a narrow single-line question (default for type A), inline only for one-off simple ops. Build the prompt by the "Direction line template" from main.md; put the resolved Ambient block at the start (`inc-rule-subagent-run`).

## Templates

- Artifacts: `inceptions/templates/motivation.md`, `overview.md`, `technical-task.md`, `result.md`, `rule.md`.
- Subagent responses: `inceptions/templates/executor-report.md`, `researcher-report.md`, `reviewer-report.md`, `final-report.md`.
