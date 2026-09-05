# Technical task: try-{N}-{Level} — {Title}

## Technical task (верхнеуровневое ТЗ)
{Техническое задание, сформированное инженером из исследования.}

## Tasks
- {N}-{description} — {суть задачи} — suggested agent: {inc-executor} — used agent: {…}
- ...

## Execution scheme
> Каждый id задачи — имя файла задачи (например, `1-abstractions`).
> MANDATORY! Каждая задача выполняется отдельным субагентом `inc-executor` (Task tool). Не выполнять inline. Без исключений — даже если задача кажется тривиальной.
- Phase 1 (sequential): task {N}-{description} → task {N}-{description}
- Phase 2 (parallel):   task {N}-{description} || task {N}-{description}
- Phase 3 (sequential): task review — проверить все изменения, исправить несоответствия

## Execution mode
{[A] automatic | [B] step by step | [C] inline}
