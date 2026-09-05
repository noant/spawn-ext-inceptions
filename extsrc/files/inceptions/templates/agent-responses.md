# Шаблоны ответов субагентов (Output format)

Шаблоны ответов, которые каждый ролевой субагент заполняет в своём финальном ответе родителю. Ссылаться на них из шаблонов запросов в `inceptions/main.md`.

## Отчёт исполнителя (inc-executor)

```markdown
# Report: Task {N} — {Short title}

## Status
{Done | Partial | Failed}

## What was done
{Что сделано — конкретно, по пунктам.}

## What was not done
{Что не сделано и почему.}

## Artifacts
{Список созданных/изменённых файлов или результатов.}

## Problems / blockers
{Возникшие проблемы, блокеры.}

## Questions
{Вопросы к инженеру, если остались.}

## Model
My model: {model}
```

## Отчёт исследователя (inc-explorer / inc-researcher)

```markdown
# Research report: {направление / тема}

## Direction
{Линия направления, которая была задана, или "free".}

## Research file
{Ссылка на файл исследования: res/{task-descr-slug}.{agent-slug}.md}

## Summary
{Краткая сводка (2-5 предложений) — что выяснено.}

## Gaps
{Что не удалось выяснить, что требует уточнения.}

## Model
My model: {model}
```

## Отчёт ревьюера (inc-reviewer)

```markdown
# Review report: {объект ревью}

## Object
{Что ревьюилось: исследование | задачи | выполненная работа.}

## Verdict
{Approved | Needs fixes | Rejected}

## Issues
### Critical
- {проблема} — {где} — {почему критично}

### Minor
- {проблема} — {где}

## Suggestions
{Рекомендации по улучшению.}

## Follow-up
{Нужны ли дополнительные исследования (вызов inc-explorer или inc-researcher).}

## Model
My model: {model}
```

## Финальный отчёт пользователю

```markdown
# Final report: {Inception {N} — {Title}}

## Outcome
{Success | Partial | Failed}

## What was achieved
{Что удалось — кратко.}

## Why (причины результата)
{Причины успеха или неуспеха.}

## Highlighted problems
{Подсвеченные проблемы, которые всплыли.}

## Next step proposal
{Предложение продолжить цикл в рамках следующей задачи — с учётом предыдущей работы и контекстом ко всем шагам и проблемам.}

## Extracted rules
{Извлечённые правила inc-rule-{slug}, если есть.}
```
