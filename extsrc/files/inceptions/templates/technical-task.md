# Technical task: try-{N}-{Level} — {Title}

## Technical task (high-level spec)
{Technical task formed by the engineer from the research.}

## Tasks
- {N}-{description} — {task essence} — suggested agent: {inc-executor} — used agent: {…}
- ...

## Execution scheme
> Each task id is the task file name (e.g., `1-abstractions`).
> MANDATORY! Each task is performed by a separate `inc-executor` subagent (Task tool). Do not execute inline. No exceptions — even if the task seems trivial.
- Phase 1 (sequential): task {N}-{description} → task {N}-{description}
- Phase 2 (parallel):   task {N}-{description} || task {N}-{description}
- Phase 3 (sequential): task review — check all changes, fix inconsistencies

## Execution mode
{[A] automatic | [B] step by step | [C] inline}
