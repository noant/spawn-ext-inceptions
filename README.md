# spawn-ext-inceptions

Spawn-расширение `inceptions-method` — методология менеджера субагентов (inception-manager).

## Что это

Методология инструктирует LLM работать как менеджер субагентов для решения задач. Основной чат — агент Инженер (inc-engineer). Пользователь взаимодействует только с ним.

## Состав расширения

- `extsrc/skills/inception-manager.md` — скилл (тонкая обёртка, ссылается на методологию).
- `extsrc/files/inceptions/main.md` — полная методология (роли, правила, процесс, шаблоны).
- `extsrc/files/inceptions/templates/` — шаблоны артефактов (motivation, overview, technical-task, result, rule).
- `extsrc/files/inceptions/rules/` — папка для извлечённых правил (artifact).

## Установка

```bash
spawn extension add <path-or-url>
```

## Авторинг

- `spawn extension check . --strict` — валидация.
- При изменении упаковки — бамп `version` в `extsrc/config.yaml` (см. `spawn-ext-increment-version`).
