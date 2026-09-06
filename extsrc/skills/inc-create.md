---
name: inc-create
description: Entry point of the inceptions methodology. Request classification (question/initiative), clarification, creation of inception and attempt (motivation.md, overview.md, res/).
---

# inc-create

Entry point of the inceptions methodology. Role: `IA1-drafter` (main chat).

Full methodology: `inceptions/main.md`. Follow it.

## What it does

Classifies the user request and routes it: answers a question (light mode), or drafts a spec and creates an inception/attempt (Step 1), or continues an existing inception.

## Order

1. Read `inceptions/main.md` fully.
2. Classify the request (Step 0) per `IR16-classify`: **[A] Question** (asks) or **[B] Initiative/task** (explicit "do" / research).
3. If unclear — ask clarification (`IR9-ask`).
4. If the request relates to an existing inception in `inceptions/` — propose to continue (use `inc-continue` to determine where).
5. **[A] Question** → answer directly (light mode, no `inceptions/` artifacts); optionally launch `IA2-researcher`/`IA3-explorer` for read-only facts. If the answer reveals a real task, propose Step 1.
6. **[B] Initiative/task** → proceed to Step 1 (Spec drafting). Follow the full flow in main.md: research (1.0), navigation (1.1), clarifications (1.2), overview (1.3), decomposition (1.4).

Steps 5 and 6 are branches, not a sequence — pick the one matching the request type.

## Launching a research subagent

When research is needed (type A answer; type B research loop), launch a subagent via the task tool. Role choice: `IA3-explorer` for decomposition into independent lines, `IA2-researcher` for a narrow single-line question (default for type A), inline only for one-off simple ops. Build the prompt per **Subagent run protocol** (`IR15-ambient`), then the line from `IR12-model-line`.

## Templates

- Artifacts: `inceptions/templates/motivation.md`, `overview.md`, `subtask.md`, `research.md`, `rule.md`.
- Subagent responses: `inceptions/templates/executor-report.md`, `researcher-report.md`, `reviewer-report.md`, `final-report.md`.
