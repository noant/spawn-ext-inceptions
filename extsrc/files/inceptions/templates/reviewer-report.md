# Reviewer report (inc-reviewer)

Response template (Output format) that the `inc-reviewer` subagent fills in its final response to the parent.

```markdown
# Review report: {review object}

## Object
{What was reviewed: research | spec | completed work.}

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
