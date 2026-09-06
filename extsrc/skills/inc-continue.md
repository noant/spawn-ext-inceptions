---
name: inc-continue
description: Determines where to continue work within an existing inception/attempt. Clarification of the active attempt, reading overview/subtasks, determining the current stage and next step.
---

# inc-continue

Determines where to continue work within an existing inception. Role: `IA1-drafter` (main chat).

Full methodology: `inceptions/main.md`. Follow it.

## What it does

When the user wants to continue work on an existing inception — determines the active attempt, the current stage, and the next step.

## Clarification of the active attempt

1. Determine which inception/attempt folder to continue working on:
   - If the active attempt is known from context — use it.
   - If unclear — ask the user (`IR9-ask`) which inception/attempt to continue.
2. Read the attempt's `overview.md` (stage statuses, goal, research summary, subtasks).
3. Read the subtask files `{N}-{description}.md` (if present) — status, suggested/used model.
4. Read the inception's `motivation.md` (if needed) — status, attempts.

## Determining the current stage

By the `[V]` statuses in `overview.md`, determine at which stage the work stopped:

- No `[V] Research` → continue from Step 1.0 (Research).
- Has `[V] Research`, no `[V] User research review` → continue from Step 1.0.3 (User research review).
- Has `[V] User research review`, no `[V] Spec drafting` → continue from Step 1.3 (Spec drafting).
- Has `[V] Spec drafting`, no `[V] Spec self-review` → continue from Step 2 (Spec self-review).
- Has `[V] Spec self-review`, no `[V] Spec review` → continue from Step 3 (Spec review).
- Has `[V] Spec review`, no `[V] Execution` → continue from Step 4 (Execution).
- Has `[V] Execution`, no `[V] Execution self-review` → continue from Step 5 (Execution self-review).
- Has `[V] Execution self-review`, no `[V] Result review` → continue from Step 6 (Result review).
- Has `[V] Result review`, no `[V] Documentation update` → continue from Step 7 (Documentation update).
- Everything marked → attempt is closed, propose a new attempt or a new inception.

## Order

1. Read `inceptions/main.md` fully.
2. Clarify the active attempt (see above).
3. Determine the current stage by statuses (see above).
4. Tell the user where the work stopped and what is proposed next.
5. Propose to continue from the needed step (per `inceptions/main.md`).

## Templates

- Artifacts: `inceptions/templates/overview.md`, `subtask.md`, `motivation.md`.
- Subagent responses: `inceptions/templates/executor-report.md`, `researcher-report.md`, `reviewer-report.md`, `final-report.md`.
