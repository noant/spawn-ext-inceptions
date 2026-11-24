---
name: inc-research
description: Standalone research via the inceptions research methodology — fact-lists with sources, no full inception process. Use for "research this", "find out", "compare", "collect facts" requests that do not need spec drafting or execution.
---

# inc-research

Standalone research entry point of the `inceptions` extension. Pure research without inceptions artifacts (`spawn/inceptions/` folder, overview, subtasks) and without the full process.

Research methodology: `spawn/inceptions/research.md`. Read it fully and follow it in **standalone mode**.

## What it does

Answers a research request with a fact-list (facts | source | confidence, gaps, sources) collected inline or via research subagents, and produces the deliverable (note/tutorial/book) per sections 3-7 of `spawn/inceptions/research.md`.

## When to use

- The user asks to research/find/compare/collect facts — and wants an answer, not a deliverable production task.
- If the request reveals a real production task (document, plan, implementation) — propose the full methodology (`inc-create`) instead.

## Order

1. Read `spawn/inceptions/research.md` fully.
2. Clarify if needed: goal, output format (note/tutorial/book), depth (inline/medium/high), audience, scope boundaries.
3. Run the stages E0-E7 (standalone mode): plan -> researchers -> review -> writers -> review -> assembly.
4. Subagent launches: Subagent run protocol (`IR15-ambient`, `IR12-model-line` from `spawn/inceptions/main.md`); researchers are read-only.
5. Output: final response with the merged fact-list (Facts / Gaps / Sources) and the quality checklist applied.
6. Placement: results go to the `researches/` folder per section 8 of `spawn/inceptions/research.md` (single: `researches/{slug}.md`; multi-part: `researches/{Title}/` with `00-overview.md`); intermediate fact-lists to `_research/` inside the work folder.

## Limits

- No `spawn/inceptions/` attempt artifacts are created (no inception/attempt folders).
- Max 3 waves, max 4 search iterations per question, gaps recorded honestly (all per `spawn/inceptions/research.md`).
