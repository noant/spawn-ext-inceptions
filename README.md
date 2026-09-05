# spawn-ext-inceptions

Spawn extension `inceptions-method` — subagent manager methodology (inception-manager).

## What it is

The methodology instructs the LLM to work as a subagent manager to solve tasks. The main chat is the Engineer agent (inc-engineer). The user interacts only with it.

## Extension composition

- `extsrc/skills/inc-create.md` — entry point skill (request classification, language detection, inception/attempt creation).
- `extsrc/skills/inc-continue.md` — determines where to continue work within an existing inception.
- `extsrc/files/inceptions/main.md` — full methodology (roles, rules, process, templates).
- `extsrc/files/inceptions/templates/` — artifact templates (motivation, overview, technical-task, result, rule, agent-responses).
- Extracted inception rules are stored in `spawn/rules/` (artifact, standard Spawn mechanism).

## Installation

```bash
spawn extension add <path-or-url>
```

## Authoring

- `spawn extension check . --strict` — validation.
- When changing the packaging — bump `version` in `extsrc/config.yaml` (see `spawn-ext-increment-version`).
