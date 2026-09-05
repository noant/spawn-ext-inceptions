---
name: inc-extract-rules
description: Этап 4a: экстракт правил в spawn/rules/ и spawn refresh. Кларификация активной попытки.
---

# inc-extract-rules

Этап 4a: обновление документации и экстракт правил. Роль: `inc-engineer` (основной чат).

Полная методология: `inceptions/main.md`. Следуй ей.

## Кларификация активной попытки

Сначала определить, над какой папкой inc'а продолжаем работу:
- Если активная попытка известна из контекста — использовать её.
- Если неясно — спросить пользователя (`inc-rule-ask`).
- Прочитать `overview.md`, `technical-task.md`, `result.md` этой попытки.

## Порядок (Этап 4a)

1. Прочитать `inceptions/main.md` полностью.
2. Кларифицировать активную попытку (см. выше).
3. Извлечь правила из выполненной работы (`inc-rule-extract`).
4. Сохранить каждое правило в `spawn/rules/{SLUG}-{N}-{description}.md` — одно правило на файл.
5. Обновить документацию (overview, technical-task, result, motivation при закрытии).
6. Вызвать `spawn refresh` для перерендера скиллов и обновления `spawn/navigation.yaml`.

## Шаблоны

- Файл правила: `inceptions/templates/rule.md`.
