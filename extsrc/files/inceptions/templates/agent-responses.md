# Subagent response templates (Output format)

Response templates that each role subagent fills in its final response to the parent. Reference them from the request templates in `inceptions/main.md`.

## Executor report (inc-executor)

```markdown
# Report: Task {N} — {Short title}

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

## Researcher report (inc-explorer / inc-researcher)

```markdown
# Research report: {direction / topic}

## Direction
{The direction line that was set, or "free".}

## Research file
{Link to the research file: res/{task-descr-slug}.{agent-slug}.md}

## Summary
{Brief summary (2-5 sentences) — what was found.}

## Gaps
{What could not be found, what requires clarification.}

## Model
My model: {model}
```

## Reviewer report (inc-reviewer)

```markdown
# Review report: {review object}

## Object
{What was reviewed: research | tasks | completed work.}

## Verdict
{Approved | Needs fixes | Rejected}

## Issues
### Critical
- {problem} — {where} — {why critical}

### Minor
- {problem} — {where}

## Suggestions
{Recommendations for improvement.}

## Follow-up
{Whether additional research is needed (calling inc-explorer or inc-researcher).}

## Model
My model: {model}
```

## Final report to the user

```markdown
# Final report: {Inception {N} — {Title}}

## Outcome
{Success | Partial | Failed}

## What was achieved
{What was achieved — briefly.}

## Why (reasons for the result)
{Reasons for success or failure.}

## Highlighted problems
{Highlighted problems that surfaced.}

## Next step proposal
{Proposal to continue the cycle in the next task — taking into account the previous work and context of all steps and problems.}

## Extracted rules
{Extracted inc-rule-{slug} rules, if any.}
```
