# Inceptions: Research Methodology

Unified research methodology of the `inceptions` extension: research and production of educational content (tutorial, guide, textbook, book, monograph, handbook, long note). Full transfer and superset of the former `research-writing` approach.

Two usage modes:

- **Full mode** — Step 1.0 of `spawn/inceptions/main.md` (research before spec drafting) and type [A] question answering (Step 0.1). Research files go to the attempt folder `res/`.
- **Standalone mode** — via the `inc-research` skill: pure research without the full inceptions process. Results go to the `researches/` folder.

Relation to other skills:
- `research-note-create` — short note; this methodology is its generalization.
- `book-create` — book production; this methodology is the superset.
If the request is explicitly about a book — `book-create` can be used; otherwise this methodology.

## 1. Input (parameters)

1. Task and goal: why the reader needs the result (learning, reference, problem solving).
2. Output format:
   - `note` — one note/overview;
   - `tutorial` — tutorial/guide with practice;
   - `book` — multi-part work (textbook, monograph, reference).
3. Language (`language`, default Russian).
4. Audience and starting level: `beginner` / `developer` / `expert`.
5. Research depth: `docs-only` (documentation services) / `web` (+ web search) / `deep` (+ repository examples).
6. Theory/practice ratio: `theory-heavy` / `balanced` / `practice-first`.
7. Context binding: `generic` or project-bound (read hla-files and notes before research).
8. Methodic flags (for tutorial/book): exercises, control questions, capstone project, mastery checklist, glossary.
9. Production process: `once` / `staged` (in parts with user review; default for book).

If a parameter is not specified and it affects the result — ask the user (`IR9-ask` in full mode; direct question in standalone mode). Decide minor defaults yourself; do not flood with questions.

## 2. Research methodology

### 2.1 Principles

- A fact without a source is not a fact: mark it "unverified" or do not state it at all.
- Freshness: check version/API currency (a two-year-old article may describe a dead API).
- Primary source over retelling: official docs and repositories > blogs > aggregators.
- Stop rule: after 4-5 consecutive failed searches stop and record the gap; do not loop.
- Honesty about gaps: what was not found is recorded explicitly; no invention.

### 2.2 Source priority

1. Library/framework documentation services — curated API docs for known technologies.
2. Official documentation and changelogs (direct page fetching).
3. Repositories (README/code fetching, code examples).
4. Web search (multi-engine) — fresh guides, comparisons, known issues.
5. Secondary sources (articles, Q&A sites) — only with cross-checking.

Workspace and git repositories count as primary sources for project-internal questions (workspace document reads/search, repository file reads/search).

### 2.3 Fact-list (format)

Every research output (research file and researcher report summary) uses:

```
## Facts
- {fact} | source: {url} | confidence: high/medium
## Gaps
- {what could not be found — honestly}
## Code examples
- {ready-to-use examples with attribution}
## Sources
- {list of urls}
```

- Facts without a source are marked "unverified".
- Code examples usable as-is go into the research file with attribution.

### 2.4 Limits

- No more than 4 search iterations per question.
- A gap -> at most 2 targeted follow-up reads, then the gap is fixed as final.
- Research loop: at most 3 waves (full mode Step 1.0.2).

## 3. Educational text writing methodology

### 3.1 Principles

- From the reader, not the author: introduce every concept before its first use.
- One step — one idea. Do not mix concept, syntax, and best practice in one paragraph.
- Before writing: formulate what the reader CAN DO after reading (learning outcomes). If it cannot be formulated — the section is not needed.
- Practice is mandatory: at least one working code example per section; examples must be compilable/runnable, not pseudocode.
- Progression: complex builds on simple; each next section relies on the previous ones.
- Mistakes are part of learning: show typical mistakes and how to diagnose them.

### 3.2 Skeletons by format

- All formats: the document starts with a "Dictionary of terms and definitions" block (before the main text).
- Every dictionary term: term -> definition (1-2 sentences) -> term source
  (one of: "our own term" / "domain term" / "term of technology {X}" with the technology named / "term of standard {X}").
  Example table row: `Execution item | A process step performed by a human or a service | our own term`.
  If the term is external — the source is mandatory (which technology/standard defines it, link to docs).
- `note`: dictionary -> what it is and why -> core concepts -> layers/components (table) -> integration points -> typical end-to-end flow -> mistakes -> recommendations -> section dictionary (new terms) -> sources.
- `tutorial`: dictionary -> motivation -> concepts -> step-by-step practice with code -> exercises -> summary.
- `book`/`tutorial-genre`: like tutorial, chapters per plan; dictionary — in the book introduction and at the start of each chapter (only new terms).
- `monograph`: dictionary -> problem statement -> state of the art / solution comparison -> detailed analysis -> applicability boundaries -> conclusions.
- `handbook`: dictionary -> definition -> syntax/properties/parameters (tables) -> typical recipes -> mistakes and edge cases.

Mandatory elements of any section: at least one code example; at least one diagram/table (if applicable); links to sources; new terms into the glossary.

### 3.3 Detail levels by audience

- `beginner`: full context, extensive glossary, every action spelled out, no "obviously".
- `developer`: common practices assumed; focus on differences and integration.
- `expert`: concise, essence only, trade-offs and edge cases; minimal introduction.

## 4. Subagent scheme

Role of the current chat: coordinator. Subagents via the task tool. Maximum 2 nesting levels; subagents are forbidden to launch their own subagents.

### 4.1 Roles and models

| Role | Purpose | Model |
|------|---------|-------|
| `planner` | Structure: sections/chapters, goals, learning outcomes, dependencies, key questions, what to read | strong (GLM-5.3 or deepseek-v4-pro) |
| `researcher` (N, parallel) | Fact collection per topic/section; output — fact-list | fast (deepseek-v4-flash; fallback glm-5.3-flash) |
| `writer` (N) | Section draft from the fact-list | medium/strong (deepseek-v4-pro or GLM-5.3) |
| `reviewer` | Review: facts, style, terminology consistency, mandatory elements | GLM (glm-5.3 / glm-5.3-flash) or deepseek-v4-pro |
| `assembler` | Assembly, table of contents, cross-section terminology, glossary, consolidated source list | fast |

Mapping onto inceptions roles (full mode):
- `planner` = `IA1-drafter` (Step 1: research loop decomposition, overview, subtasks)
- `researcher` = `IA2-researcher` / `IA3-explorer` (fact-lists into `res/`)
- `writer` = `IA5-executor` (one per subtask)
- `reviewer` = `IA4-reviewer`
- `assembler` = `IA6-coordinator` (aggregation, final report)

For a short task (`note`, `once`) the scheme compresses: the coordinator plans itself, 1-2 researchers, writer, reviewer — or everything inline if the volume is small.

### 4.2 Stages

```
E0. Parameters and context (coordinator, inline)
E1. Plan (planner): sections, goals, learning outcomes, key questions, what to read.
    Plan review by the coordinator; with staged — user approval.
E2. Research (researcher x N, parallel) -> fact-lists.
E3. Research review (coordinator/reviewer): gaps -> targeted follow-up (max 2 iterations).
E4. Drafts (writer x N): fact-list + formatting rules + format skeleton.
E5. Draft review (reviewer): checklists from section 6.
E6. Rework and assembly (writer/assembler).
E7. Acceptance (coordinator -> user): show the result to the user via the platform's navigation tool; with staged — next step is the user's choice.
```

In full mode E1-E3 correspond to Step 1.0 (research loop, waves up to 3), E4-E6 to Steps 4-5.

### 4.3 Ambient rules (mandatory in every subagent prompt)

```
Ambient rules:
1) Work in the docs workspace of your platform; use its document read/write tools for artifacts under {output_dir}.
2) Language: {language}. ASCII in schemas and code.
```
If the subagent works with a git repository — add the repository key and session_id. Resolve per the Subagent run protocol (`IR15-ambient`, `IR12-model-line` from `spawn/inceptions/main.md`).

## 5. Prompt templates

### 5.1 planner

```
You are the planner for "{title}" ({format}, audience: {audience}, language: {language}).
Goal of the work: {goal}. Project context: {context_summary}.
Output:
1. List of sections/chapters in reading order.
2. For each: goal, learning outcomes (what the reader can do after), volume (words),
   key research questions (5-10), what to read (sources/project files).
3. Dependencies between sections (linear/modular).
4. Overly broad topics — split into 2 independent blocks for parallel researchers.
Do not write the text itself, only the plan.
```

### 5.2 researcher

```
You are the researcher for "{title}". Topic/section: {section_title}.
Questions to cover: {bullet list from the plan}.
Tools: use the search tooling available on your platform — documentation services, web search, page/repository fetching.
Limit: no more than 4 search iterations per question; after 4-5 consecutive failed searches — record the gap and stop.

{Ambient rules}

Output (strict):
## Facts
- {fact} | source: {url} | confidence: high/medium
## Gaps
- {what could not be found — mark honestly, do not invent}
## Code examples
- {ready-to-use examples with attribution}
## Sources
- {list of urls}
No speculation. Facts without a source are marked "unverified".
```

### 5.3 writer

```
You are the writer for "{title}". Section: {section_title}.
Audience: {audience}. Language: {language}. Format skeleton: {skeleton}.
Fact-list: {researcher_output}.
Rules:
- Use ONLY facts from the fact-list; unverified — mark it.
- Every section: at least 1 working code example, 1 table/diagram (if applicable), links to sources.
- At the start of the document — a "Dictionary of terms" block: term, definition, term source
  (our own / domain term / term of technology {X} / term of standard {X}).
- New terms go into the "Section dictionary" block.
- Short sentences, lists; no emoji (only warning and colored circles/checkmarks).
- ASCII in schemas and code; code blocks with the language tag.
{Ambient rules}
Output: ready markdown of the section + "Section dictionary" block + "Sources" block.
```

### 5.4 reviewer

```
You are the reviewer for "{title}". Input: {draft} (+ fact-list).
Check and return a report:
1. Facts: is every statement confirmed by the fact-list? List of discrepancies.
2. Method: are concepts introduced before use; are mandatory elements present
   (code example, table/diagram, sources, dictionary).
3. Style: short sentences, lists, no emoji, ASCII schemas, code block languages.
4. Consistency: terminology matches other sections ({terms}).
5. Gaps: what is marked "unverified" — acceptable or blocking.
Verdict: ready / needs rework (with a list of concrete edits per point).
```

## 6. Quality checklists

### 6.1 Research (before E4)

- [ ] Every key question of the plan is covered by a fact or an honest gap.
- [ ] Sources are fresh (versions/currency checked).
- [ ] Primary sources prioritized over retellings.
- [ ] Code examples taken from docs/repos or written against verified APIs.
- [ ] Gaps recorded explicitly, no speculation.

### 6.2 Section draft (E5)

- [ ] The document starts with a "Dictionary of terms": term, definition, term source (our own / domain / technology / standard).
- [ ] Learning outcomes of the section are achieved.
- [ ] All concepts introduced before first use.
- [ ] At least 1 working code example (not pseudocode).
- [ ] At least 1 diagram/table (if applicable).
- [ ] Sources listed; "unverified" marked.
- [ ] Dictionary of new terms.
- [ ] Typical reader mistakes shown (for tutorial).
- [ ] Style: short sentences, lists, no emoji, ASCII.

### 6.3 Assembly (E6)

- [ ] Terminology uniform across all sections.
- [ ] Consolidated glossary assembled, no duplicates.
- [ ] Consolidated source list, no broken links.
- [ ] Cross-references between sections are correct.
- [ ] Chapter statuses updated ([plan] -> [draft] -> [review] -> [done]).

### 6.4 Mastery checklist (for the reader, methodic flag)

- [ ] I can explain in my own words why {X} is needed.
- [ ] I can run the example from the text without edits.
- [ ] I can modify the example for my task.
- [ ] I know the typical mistakes and how to diagnose them.

## 7. Formatting rules

- Text language is the `language` parameter (default Russian); no emoji (only warning sign and colored circles/checkmarks).
- Style: short sentences, lists; recommendations with pros/cons justification.
- ASCII in schemas and code; code blocks with the language tag; `csharp`/`javascript` tags forbidden in Jira context (see the Jira instruction if applicable).
- No lorem ipsum and no stubs.

## 8. Result placement

- **Full mode (within the inceptions process):** research files go to the attempt folder `spawn/inceptions/{N}-{inception-slug}/{N}-{description}/res/{research}.md`; deliverables are produced by executors per the subtask list.
- **Standalone mode (inc-research, no inceptions process):**
  - `note` / single research result: `researches/{slug}.md`
  - multi-part works: `researches/{Title}/` with `00-overview.md` (plan and statuses) and `{NN}-{chapter-slug}.md` chapters
  - intermediate fact-lists and reviews: `_research/` inside the work folder (to continue later)
  - ask the user for the docs root if it is not clear from context; do not guess silently.

## 9. Output

- Documents are written via the platform's document write tool (or the executor protocol in full mode); statuses reflected in `00-overview.md` (for book).
- Show the finished result to the user via the platform's navigation tool.
- With `staged` — after each part ask the user: continue / edits / change focus.
- Offer (do not perform voluntarily) pushing into a git notes repository if one exists.
