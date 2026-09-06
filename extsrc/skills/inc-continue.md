---
name: inc-continue
description: Determines where to continue work within an existing inception/attempt. Clarification of the active attempt, reading overview/technical-task/result, determining the current stage and next step.
---

# inc-continue

Determines where to continue work within an existing inception. Role: `inc-engineer` (main chat).

Full methodology: `inceptions/main.md`. Follow it.

## What it does

When the user wants to continue work on an existing inception — determines the active attempt, the current stage, and the next step.

## Clarification of the active attempt

1. Determine which inc folder to continue working on:
   - If the active attempt is known from context — use it.
   - If unclear — ask the user (`inc-rule-ask`) which inception/attempt to continue.
2. Read the attempt's `overview.md` (stage statuses, goal, research summary).
3. Read `technical-task.md` (if present) — tasks, execution scheme, execution mode.
4. Read `result.md` (if present) — outcome and notes.
5. Read the inception's `motivation.md` (if needed) — status, attempts.

## Determining the current stage

By the `[V]` statuses in `overview.md`, determine at which stage the work stopped:

- No `[V] Research` → continue from Stage 2 (Research).
- Has `[V] Research`, no `[V] User research review` → continue from Stage 2.6 (User research review).
- Has `[V] User research review`, no `[V] Task creation` → continue from Stage 3 (Task creation).
- Has `[V] Task creation`, no `[V] User task review` → continue from Stage 3.7 (User task review).
- Has `[V] User task review`, no `[V] Execution` → continue from Stage 4 (Execution).
- Has `[V] Execution`, no `[V] User result review` → continue from Stage 4.6 (User result review).
- Has `[V] User result review`, no `[V] Final report` → continue from Stage 4a/5 (Closing).
- Everything marked → inception is closed, propose a new attempt or a new inception.

## Order

1. Read `inceptions/main.md` fully.
2. Clarify the active attempt (see above).
3. Determine the current stage by statuses (see above).
4. Tell the user where the work stopped and what is proposed next.
5. Propose to continue from the needed stage (per `inceptions/main.md`).

## Templates

- Artifacts: `inceptions/templates/overview.md`, `technical-task.md`, `result.md`, `motivation.md`.
- Subagent responses: `inceptions/templates/executor-report.md`, `researcher-report.md`, `reviewer-report.md`, `final-report.md`.
