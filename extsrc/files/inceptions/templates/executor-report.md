# Executor report (inc-executor)

Response template (Output format) that the `inc-executor` subagent fills in its final response to the parent.

```markdown
# Report: Subtask {N} — {Short title}

## Status
{Done | Partial | Failed}

## What was done
{What was done — concretely, by items.}

## What was not done
{What was not done and why.}

## Artifacts
{List of created/changed files or results.}

## Problems / blockers
{Problems encountered, blockers.}

## Questions
{Questions to the engineer, if any remain.}

## Model
My model: {model}
```
