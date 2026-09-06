# spawn-ext-inceptions

Spawn extension `inceptions-method` — subagent manager methodology for non-coding work (research, documentation, planning, analysis, writing).

## What it is

The methodology instructs the LLM to work as a subagent manager to solve non-coding tasks. The main chat is the Engineer agent (`A1-drafter`). The user interacts only with it.

## Extension composition

- `extsrc/skills/inc-create.md` — entry point skill (request classification, inception/attempt creation).
- `extsrc/skills/inc-continue.md` — determines where to continue work within an existing inception.
- `extsrc/files/inceptions/main.md` — full methodology (roles, rules, process, templates).
- `extsrc/files/inceptions/templates/` — artifact templates (motivation, overview, subtask, research, rule, executor-report, researcher-report, reviewer-report, final-report).
- Extracted inception rules are stored in `spawn/rules/` (artifact, standard Spawn mechanism).

## Structure

```
inceptions/
  {N}-{inception-slug}/          — inception
    motivation.md                — inception motivation (strict template)
    {N}-{description}/           — attempt
      overview.md                — attempt overview (statuses, goal, motivation, subtasks, outcome)
      {N}-{description}.md       — subtask files (one per subtask)
      res/{research}.md          — research files
```

## Installation

```bash
spawn extension add <path-or-url>
```

## Authoring

- `spawn extension check . --strict` — validation.
- When changing the packaging — bump `version` in `extsrc/config.yaml` (see `spawn-ext-increment-version`).
