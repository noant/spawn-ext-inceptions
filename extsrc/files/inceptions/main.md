# Inceptions: AI-Oriented Methodology for Non-Coding Work

The `inceptions` methodology instructs the LLM to work as a subagent manager to solve non-coding tasks: research, documentation, planning, and other work. The main chat is the Engineer agent (`A1-drafter`); the user interacts only with it.

## Folder Structure

- `inceptions/main.md` — this file.
- `inceptions/templates/` — artifact and response templates (see `## Templates`).
- `inceptions/{N}-{inception-slug}/` — inception folder. `{N}` is the inception number; `{inception-slug}` is a short descriptive name.
- `inceptions/{N}-{inception-slug}/motivation.md` — inception motivation (required).
- `inceptions/{N}-{inception-slug}/{N}-{description}/` — attempt folder. `{N}` is the attempt number within the inception; `{description}` is a short descriptive name.
- `inceptions/{N}-{inception-slug}/{N}-{description}/overview.md` — attempt overview (required).
- `inceptions/{N}-{inception-slug}/{N}-{description}/{N}-{description}.md` — subtask files (optional; required when the attempt has 2+ subtasks).
- `inceptions/{N}-{inception-slug}/{N}-{description}/res/{research}.md` — research files (optional; created by research subagents).

**Embedded rules:**

Every rule has a stable label `[R{n}-{slug}]`.

_Folder hygiene_

- **`R1-paths`**
  - under `inceptions/`, only paths allowed by the Folder Structure are permitted
  - no other files
- **`R2-no-clutter`**
  - do not create READMEs or other extraneous docs under `inceptions/`
  - special case of `R1-paths`

_Inception identification_

- **`R3-inception-num`**
  - for numeric `{N}` in `inceptions/{N}-{inception-slug}/`: suggest the next number (`R10-ask`)
  - wait for an explicit reply before creating the folder
  - next number = highest existing `{N}` under `inceptions/` plus 1
- **`R4-attempt-num`**
  - for numeric `{N}` in `{N}-{description}/` (attempt): next number = highest existing `{N}` under the inception plus 1
  - no external tracker keys apply; attempts are numbered sequentially within an inception

_Attempt specs_

- **`R5-new-attempt`**
  - new attempts follow Step 1 and the overview template at the end of this file
- **`R6-process`**
  - status marks `[V]` and user prompts must reproduce the wording from the corresponding Step exactly
  - Steps run in order unless the process explicitly allows otherwise
- **`R7-concrete`**
  - specs are executable edits, not intentions
  - every overview and subtask names concrete artifacts, paths, and deliverables under change
  - every Before / After pair in subtasks is a fenced minimal excerpt (real lines or the exact replacement) plus a behavior line
  - prose-only or "change X to Y" without concrete content is invalid
- **`R8-greenfield`**
  - for new artifacts, same Before/After discipline as `R7-concrete`, with two differences:
  - Before may be insertion-context only (nothing to quote)
  - After is still fenced content plus a behavior line

_Interaction and context_

- **`R9-ask`**
  - when you must ask the user (clarifications, confirmations, choices), **stop and request from the user**
  - do not continue the workflow until they answer
  - prefer the platform structured ask tool (see `R11-ask-tools`), multiple choice when possible
  - fallback order: platform tool -> **direct request to the user in your reply**
  - "Ask" / "request from the user" means only those channels
  - **never** treat asking as launching a Task / sub-agent / other agent; those tools are not ask tools
  - if no platform ask tool is available, stop, request from the user, then wait
  - do not ask when the answer is clear from context; ask only when the answer materially changes the next action
- **`R10-navigation`**
  - before drafting an attempt, open **`spawn/navigation.yaml`**
  - read all `read-required`
  - read task-relevant `read-contextual`
  - apply them in the spec
  - if the file is absent, the rule is a no-op (proceed)
  - Spec self-review re-checks compliance; violations are defects
- **`R11-ask-tools`**
  - user-question tools by platform (data for `R9-ask` only):
  - **Cursor:** AskQuestion / cursor/ask_question
  - **VS Code/Copilot:** AskQuestions / vscode_askQuestions
  - **Claude Code:** AskUserQuestion
  - **Codex:** request_user_input
  - **Other:** IDE embedded ask tool; if none, stop and ask questions in the current chat (`R9-ask`)
- **`R12-model-line`**
  - every sub-agent prompt must include this line verbatim:
    > End your final response with the line `My model: X` where X is your actual model identifier (e.g. `claude-sonnet-4-6`, `gpt-4o`) — write your actual model identifier in place of X.
  - recording the model used by a sub-agent:
    - if the platform tool lets you pass an explicit sub-agent `model`, record that call parameter
    - if there is no model-selection parameter, read `My model:` from the sub-agent response and record that
  - who writes `Used model` / `[model-name]`: see `R14-done-marking`
- **`R13-changed-files`**
  - after finishing a create/edit batch, list every created or edited path (repo-relative, complete, no omissions)
  - renames and deletes count
  - applies to Step 1 (spec), Steps 4–5 (execution), and any user-requested edits
  - **Propagation:** the executor sub-agent includes the full list in its final response
  - the coordinator (`A6-coordinator`) aggregates lists from child sub-agents and forwards the complete set to the user (and to its parent when the coordinator itself is a sub-agent)
  - do not drop or summarize away paths
- **`R14-done-marking`**
  - after an Executor (`A5-executor`) replies, the coordinator (`A6-coordinator`) marks the subtask done:
    - rename: `{N}-{description}.md` → `_DONE_{N}-{description}.md`
    - set: `Used model: {model}` from `R12-model-line`; `Suggested model` unchanged
  - only the coordinator (`A6-coordinator`) renames and writes `Used model` — the worker (`A5-executor`) must not
  - the worker sets `Status: Done` per Step 4 Executor protocol
  - mark immediately; do not defer
- **`R15-ambient`**
  - **Ambient context** is session/environment facts for sub-agents (repository name, session, and similar) — not conventions or task design rules
  - format when present: header `Ambient rules:` then one numbered item per line (`1) …`, `2) …`):
    ```
    Ambient rules:
    1) repository: {name}
    2) session: {id}
    ```
  - explicit empty: `Ambient context: none` — ambient **is** set; do **not** ask
  - if Ambient context is missing (neither an `Ambient rules:` block nor `Ambient context: none`), the agent **must** clarify via `R9-ask` before launching any sub-agent — mandatory
  - every sub-agent launch must put the resolved Ambient block at the start of the prompt per **Subagent run protocol**
  - each agent passes the block to child sub-agents unchanged

_Subagent governance_

- **`R16-classify`**
  - at the start of any inceptions request, classify it as:
    - **[A] Question** — the user asks (wants an answer), not a "do" request
    - **[B] Initiative / task** — the user sets a task, an explicit "do", or asks to draft a spec
  - if unclear, ask via `R9-ask` and wait
  - [A] -> answer directly (light mode, no `inceptions/` artifacts); optionally launch `A2-researcher`/`A3-explorer` for read-only facts
  - [B] -> proceed to Step 1 (Spec drafting)
- **`R17-subagent-depth`**
  - limits subagent nesting — who may create whom:
  - `A1-drafter` — may launch `A2-researcher`, `A3-explorer`, `A4-reviewer`
  - `A2-researcher` — must not create subagents (works independently)
  - `A3-explorer` — may create only `A2-researcher`
  - `A4-reviewer` — may launch `A2-researcher` / `A3-explorer` (to verify findings); no deeper
  - `A5-executor` — must not create subagents (performs one subtask)
  - `A6-coordinator` — may launch `A5-executor` and `A4-reviewer`
  - violating this rule is a defect: a subagent that created a forbidden descendant must stop and return an error to the parent
- **`R18-no-ask-tool`**
  - subagents (`A2-researcher`, `A3-explorer`, `A4-reviewer`, `A5-executor`, `A6-coordinator`) are forbidden to ask questions via the ASK tool or any platform ask tool (AskQuestion, ask_question, AskUserQuestion, request_user_input, etc.)
  - a subagent asks questions only in text in its response to the parent
  - only the main chat (the agent coordinating the user, `A1-drafter` in current context) may use the ask tool for questions to the user
- **`R19-ask-if-unclear`**
  - a subagent may ask the parent a question if something in its task/direction is unclear
  - the question is asked only in text in the subagent's response (`R18-no-ask-tool`)
  - the subagent does not block indefinitely: if the answer is critical, it stops and returns the question to the parent; otherwise it proceeds with a stated assumption
- **`R20-navigate`**
  - after finishing a task/subtask, navigate the user to created/changed files via the platform navigate tool
  - navigate to each such file with a short description (chip label)
  - use line ranges (`from_line`/`to_line`) to point to the exact changed region when relevant
  - for git files pass the git session id; for ws docs omit it
  - never use navigate to read contents — only to show files in the UI

**Roles:**

Every role has a stable label `[A{n}-{slug}]`. Reference roles by label (e.g. `A1-drafter`), not by number — the label survives renumbering. One agent instance plays one role at a time.

- **`A1-drafter`** (Drafter / Engineer)
  - main chat coordinator
  - researches the subject alone or via `A2-researcher` / `A3-explorer`
  - writes the attempt specification (`overview.md`, subtasks)
  - owns Step 1 (Spec drafting); may also close Step 7 / Pattern extract in current context
- **`A2-researcher`** (Researcher)
  - general open-ended research (web, docs, workspaces, hypothesis checking)
  - read-only except the research file in `res/`
  - no subagents
- **`A3-explorer`** (Explorer)
  - coordinating researcher; decomposes research into sub-lines
  - may create only `A2-researcher`
  - supplies accurate Before / After context, paths, and artifacts
- **`A4-reviewer`** (Reviewer)
  - reviews spec / work / research
  - reviews at Spec self-review (Step 2) and Execution self-review (Step 5)
  - may launch `A2-researcher` / `A3-explorer`; no deeper
  - may fix defects found in that review; then stops and prompts the user
- **`A5-executor`** (Executor)
  - executes one subtask
  - performs that subtask only (research, docs, planning, other work); lists changed files per `R13-changed-files`
  - sets `Status: Done` in the subtask file; does not rename to `_DONE_` or write `Used model`
  - no subagents
- **`A6-coordinator`** (Coordinator)
  - coordinates Executors (`A5-executor`) per the subtask list
  - owns Steps 4–5 end-to-end: launch Executors, `R14-done-marking`, launch Reviewer for Step 5
  - may launch `A5-executor` and `A4-reviewer`
  - must not be the same agent instance as `A1-drafter`

**Subagent run protocol**

Applies to every sub-agent launch for any role (`A2-researcher`, `A3-explorer`, `A4-reviewer`, `A5-executor`, `A6-coordinator`, and any further nesting).

1. Resolve Ambient context per `R15-ambient` (clarify if missing; do not ask when `Ambient context: none`).
2. Put the resolved Ambient block at the very start of the sub-agent prompt (verbatim `Ambient rules: …` or `Ambient context: none`).
3. Then add role-specific instructions (`R12-model-line`, `R13-changed-files`, Executor protocol, etc.).
4. Every agent that received Ambient context passes it to each child sub-agent unchanged — same wording, same order; do not drop, summarize, or rewrite.

---

## Process Overview

```
[0] Request classification (Question -> answer | Initiative -> Step 1)
→ [1] Spec drafting
→ [2] Spec self-review
→ [3] Spec review (user)
→ [4] Execution
→ [5] Execution self-review
→ [6] Result review (user)
→ [7] Documentation update
→ (optional) pattern extract to spawn/rules/
```

Mark each status [V] on completion. Prompt the user after steps 2, 5, and 6. Step 0 is a gate, not a status step — it has no status checkbox. After Step 7, offer optional Pattern extract (not a Status checkbox).

---

## Step 0: Request classification

**Executor:** `A1-drafter` (current context)

0.1 Classify the request per `R16-classify`:
  - [A] Question -> answer directly (light mode); no `inceptions/` folder, no status checkboxes; optionally launch `A2-researcher`/`A3-explorer` (Subagent run protocol, `R12-model-line`) for accurate facts; if the answer reveals a real task, propose Step 1.
  - [B] Initiative / task -> proceed to Step 1 (Spec drafting).
0.2 If the type is unclear, ask via `R9-ask` and wait.

---

## Step 1: Spec drafting

**Executor:** `A1-drafter`

1.0 **Research** — run a research loop before drafting the spec when the subject is not yet clear. Skip for trivial tasks.

1.0.1 **Research depth selection** — ask the user (`R9-ask`) whether research is needed and at what depth:
  - [inline] — no subagents; the drafter researches in-chat (trivial, one-off questions)
  - [medium] — launch a single `A3-explorer` (or `A2-researcher` for a narrow single-line question) subagent (default)
  - [high] — launch several directed `A3-explorer` subagents, one per direction line
  - [skip] — no research needed; proceed directly to spec drafting
  Default to [medium] if the user does not specify. If the task is trivial and the path is obvious, offer [skip] first.

1.0.2 **Research loop** — run research in a loop, up to 3 waves:
  1.0.2.1 Decompose — split research into independent direction lines (high = several, medium = one)
  1.0.2.2 Launch — launch `A3-explorer`/`A2-researcher` subagents, one per line (Subagent run protocol `R15-ambient`, `R12-model-line`)
  1.0.2.3 Collect — record findings into the `## Research summary` section of `overview.md`; research files go to `res/{research}.md`
  1.0.2.4 Review — launch `A4-reviewer` to review the research (Subagent run protocol)
  1.0.2.5 Decide — gaps remain -> another wave (up to 3 total), return to 1.0.2.1; complete -> proceed to 1.0.3

1.0.3 **User research review** — show the research summary, ask the user (`R9-ask`):
  - [Proceed] — research sufficient, move to spec drafting
  - [Refine] — needs refinement, user points out what to clarify, return to 1.0.2 (another wave, up to 3)
  - [Stop] — stop here (finish or move to a separate chat with a prompt)

1.1 **Project rules (navigation)** — **MANDATORY!** Follow `R10-navigation` before writing any spec content.

1.2 **Implementation clarifications** — **MANDATORY!**:
- Before writing any spec content, identify ambiguous, optional, or convention-dependent aspects.
- Ask the user explicit questions (`R9-ask`) and wait for answers.
- Record answers (or agreed defaults) in **Details**.
- Skip only when there is a single obvious path.
- If **Ambient context** is not already set in this chat, ask via `R9-ask` (repository, session, and similar); accept a list or explicit `Ambient context: none` (`R15-ambient`).
- If **motivation** is unclear, ask via `R9-ask` (multiple choice; adapt to context):
  - research — gather information, answer a question, explore a topic
  - documentation — produce or update documents, guides, specs
  - planning — produce a plan, roadmap, or breakdown
  - analysis — analyze data, compare options, evaluate
  - writing — produce prose, reports, summaries
  - decision support — evaluate options and recommend
  - other — name the driver in one sentence
- Put the chosen motivation in **`## Motivation`** (after **Goal**).

1.3 **Overview**
- `inceptions/{N}-{inception-slug}/{N}-{description}/overview.md` follows the overview.md template
- sections through `## Details` (before/after and examples go there)
- **Goal** = one sentence
- **Motivation** immediately after **Goal**
- add `## Subtasks` only when work splits into 2+ subtasks

1.4 **Decomposition**
- when work has 2+ subtasks:
  - subtask ids in `## Subtasks` must match `{N}-{description}.md` filenames from 1.5
- create `{N}-{description}.md` per subtask with:
  - goal
  - approach
  - affected artifacts (named files/documents/deliverables per path)
  - changes (before/after)
- set `Suggested model` for the subtask; leave `Used model` empty
- optional: launch `A3-explorer` (or `A2-researcher`) as a new sub-agent for read-only research to determine accurate **Before** / **After** text, then merge findings into the subtask files (research only; `A1-drafter` owns decomposition and the spec); follow **Subagent run protocol** (`R15-ambient`)

- set [V] "Spec drafting" `[model-name]` (record the model per `R12-model-line`): `- [V] Spec drafting [model-name]`
- list changed files per `R13-changed-files`

---

## Step 2: Spec self-review

**Executor:** `A4-reviewer`

- sub-agent prompt: **Subagent run protocol** first (`R15-ambient`), then the line from `R12-model-line`
- review the spec for:
  - scope impact, errors, sequencing issues
  - concrete artifacts, paths, and deliverables per `R7-concrete` in every subtask and overview
  - compliance with `R10-navigation`
- fix if needed
- set [V] "Spec self-review" `[model-name]` (record the model per `R12-model-line`): `- [V] Spec self-review [model-name]`
- list changed files per `R13-changed-files` if any files were edited
- prompt: "Spec self-review complete — spec is ready for your review (Step 3). Reply 'spec review passed', 'lgtm', or 'ok' when satisfied."

---

## Step 3: Spec review

**Executor:** User

- on confirmation ("spec review passed", "lgtm", "ok"):
  - set [V] "Spec review"
  - prompt: "Reply 'implement' to start."

---

## Step 4: Execution

**Executor (coordination):** `A6-coordinator`
- **Same chat as Steps 1–2:**
  - `A1-drafter` must not act as `A6-coordinator` for Steps 4–5
  - on the implementation command, launch **one new sub-agent** as `A6-coordinator` for Steps 4–5 end-to-end
  - parent waits for the coordinator, then waits for the user for Step 6
- **Fresh execute chat** (Steps 1–2 not in context): the current agent is `A6-coordinator`

**`A6-coordinator`** — follows the subtask list, launches one `A5-executor` per subtask, then Step 5 (`A4-reviewer`).
**Each subtask:** `A5-executor` (new sub-agent) — child of the coordinator.

- on "run it" / "implement" / "execute" / any direct instruction to start execution:
  - if "Spec review" is not yet marked, set [V] "Spec review" automatically (implementation command implies approval)
  - if this chat already completed Steps 1–2 for the task:
    - launch the Steps 4–5 `A6-coordinator` sub-agent (see Executor above) and stop coordinating inline
    - include in prompt: **Subagent run protocol** (`R15-ambient`); follow Steps 4–5 for `inceptions/{N}-{inception-slug}/{N}-{description}/` as `A6-coordinator`; the line from `R12-model-line`
  - **MANDATORY!** launch an `A5-executor` sub-agent for each subtask — do NOT execute inline; no exceptions, even if a subtask seems trivial
    - prefer subtask `Suggested model`: pass as Task/sub-agent `model` when supported; else name in prompt and use nearest slug
    - Executor prompt must include: **Subagent run protocol** (`R15-ambient`), line from `R12-model-line`, changed-files list per `R13-changed-files`, **Executor protocol** below
  - execute subtasks sequentially (one after another); there is no parallel execution scheme in this methodology

- per subtask: after the Executor replies, mark done per `R14-done-marking` (rename + `Used model`)
- when all subtasks done and Step 5 complete: set [V] "Execution" `[model-name]` (coordinator model, per `R12-model-line`): `- [V] Execution [model-name]`
- forward the aggregated changed-files list to the user per `R13-changed-files`

**Executor protocol** (`A5-executor`, each subtask):
- Implement the subtask; list changed files per `R13-changed-files`.
- At the end, set `Status: Done` in the subtask file.
- End the reply with the `My model:` line from `R12-model-line`.

---

## Step 5: Execution self-review

**Executor:** `A4-reviewer` (new sub-agent; launched by `A6-coordinator`)

- sub-agent prompt: **Subagent run protocol** first (`R15-ambient`), then the line from `R12-model-line`
- review all changes: inconsistencies, naming, missing artifacts, broken references; respect Ambient context when present
- fix if needed
- set [V] "Execution self-review" `[model-name]` (record the model per `R12-model-line`): `- [V] Execution self-review [model-name]`
- list changed files per `R13-changed-files` if any files were edited
- prompt: "Self review done. Reply 'result review passed' to proceed."

---

## Step 6: Result review

**Executor:** User

- on confirmation ("result review passed", "lgtm", "ok"):
  - set [V] "Result review"
  - fill `## Outcome` in `overview.md` (Success / Partial / Failed) per the user's assessment
  - prompt: "Will now update documentation (Step 7)."

---

## Follow-up changes after execution

If the user requests rework or fixes after Step 4:

- carry out the changes (as `A5-executor` or current context); list changed files per `R13-changed-files`
- ask via `R9-ask`: "Do you want to update the specifications of the current attempt?"
  - Yes: `A1-drafter` (or current context) edits the affected subtask files and/or `overview.md` to match the actual state; do not re-run the spec cycle; list changed files per `R13-changed-files`
  - No: proceed without changes

---

## Step 7: Documentation update

**Executor:** `A1-drafter` (current context)

- do not start Step 7 until **Result review** is marked (Step 6)
- **Scope** — from subtasks and the files changed/added in this attempt, update the inception's `motivation.md` (attempts list, result) and any other affected docs
- **Write** — align the markdown with the actual state after this attempt
- set [V] "Documentation update" — fill the model name in brackets: `- [V] Documentation update [model-name]`
- list changed files per `R13-changed-files`
- continue with **Optional: Pattern extract** below (same run when closing via Steps 6–7)

---

## Optional: Pattern extract (after Step 7)

**Executor:** `A1-drafter` (current context)

After Step 7, optionally extract reusable approaches into **`spawn/rules/`** as project-standard candidates. Not a Status item.

The agent filters candidates, presents the final list to the user in this run (no preliminary per-candidate questions), and waits for one per-candidate reply.

**Order (mandatory):**

1. **Discover** — find and filter reusable candidates (agent only).
2. **Present** — show the filtered list to the user in one message right after Step 7 (title + one-line rationale + suggested scope per survivor).
3. **Wait** — for the user's per-candidate reply.
4. **Write** — only candidates the user accepted as Required or Optional.

Skip the whole step only if the user already declined in this close-out. If Discover leaves zero candidates, say so briefly and stop.

### Discover (before presenting)

- review this attempt's changes, subtasks, and relevant context for reusable patterns
- apply **Selection criteria** below; reject junk immediately
- if zero candidates remain: say so briefly and stop — do not present; do not invent fillers

### Selection criteria

Propose only candidates that pass all of:

- **Reusable** — useful beyond this single attempt
- **Actionable** — can become a short rule an agent can follow
- **Standard candidate** — plausible as a lasting convention
- **Not already covered** — check **`spawn/rules/`**, **`spawn/navigation.yaml`**, and related reads for duplicates
- **Pre-existing content OK** — a pattern already in use but not yet in rules remains valid
- **Examples** — prefer short real excerpts; prose-only when necessary

Reject immediately (do not offer):

- task-specific wiring, ticket ids, temporary workarounds
- restatements of existing rules or defaults
- vague slogans without an enforceable rule
- low-value or speculative ideas (junk)

### Present (after Discover)

- run only when Discover left one or more candidates
- present the entire list in **one message**, right after Step 7 — **no `R9-ask`, no per-candidate tool questions, no pauses**
- for each survivor: short title, one-line rationale, suggested scope (Required = `read-required`, Optional = `read-contextual`)
- end the message with a reply request: per-candidate Required/Optional/Decline, or "decline all"
- then wait — do not write rules, do not run `spawn refresh`, do not start the next task

### Apply the user's answer (after the reply)

- write only Required/Optional candidates
- if all Declined (or "decline all"): write nothing
- candidates not addressed default to **Decline** (do not invent consent)

### Write

1. Write under **`spawn/rules/`** (create the folder if missing).
2. Prefer an existing **`spawn/rules/`** file on the same topic — merge or revise. If none fits, create a new kebab-case Markdown file.
3. Prefer short examples in each rule when applicable (criterion 6).
4. Add each file to **`spawn/navigation.yaml`** under **`read-required` → `rules`** or **`read-contextual` → `rules`** as the user chose. Row: **`path`** + short **`description`**. Never list the same path in both.
5. Run exactly **`spawn refresh`** in the terminal.

---

## overview.md Template

```markdown
# Attempt {N}: {Title}

## Source
- Inception: {inception path or none}

## Status
- [ ] Research [model]
- [ ] Research review [model]
- [ ] User research review
- [ ] Spec drafting [model]
- [ ] Spec self-review [model]
- [ ] Spec review
- [ ] Execution [model]
- [ ] Execution self-review [model]
- [ ] Result review
- [ ] Documentation update [model]

## Outcome
{Success | Partial | Failed — filled at Step 6 after the user's assessment.}

## Goal
{One concise sentence.}

## Motivation
{Why this attempt — from the user request, or from the clarification answer.}

## Research summary
{Synthesized research findings with links to research files in res/. Present only when the research phase (Step 1.0) ran.}

## Subtasks
{List of subtask files. Present only when the attempt has 2+ subtasks.}
- {N}-{description}.md — {short description}

## Before → After
### Before
- {current state}
### After
- {desired state}

## Details
{Clarifying details, examples, constraints.}
```

Omit `## Subtasks` if there is no decomposition (single-file attempt).

---

## Subtask file template

Filename must match the subtask id from `## Subtasks` (e.g. `1-research-sources.md`). One file per subtask.

````markdown
# Subtask {N}: {Short title}

Status: Not implemented
Suggested model: {model}
Used model:

## Goal
{One sentence — outcome of this subtask.}

## Approach
{Order of work, constraints, references to research if needed.}

## Affected artifacts
- `{path/relative/to/inception/root}`
- `{...}` — {...}

## Changes (before / after)

### `{path/to/artifact.ext}` — {path plus named deliverable + what changes}

**Before**
```text
{concrete minimal excerpt or exact lines, not vague prose}
```
{what this content is — behavior, not a repeat of the diff}

**After**
```text
{replacement or new block — one-to-one with Before when editing existing text}
```
{what the new content is — behavior, not a repeat of the diff}

## Additional actions
{Optional: shell commands, manual verification steps, follow-up tasks, or other non–file-edit work for this subtask.}
````

---

## motivation.md Template

```markdown
# Inception {N}: {Title}

## Motivation
{Main motivation — why this inception, what problem it solves. 1-3 sentences.}

## Start date
{YYYY-MM-DD}

## Status
- [ ] In progress
- [ ] Closed

## Attempts
- {N}-{description} — {short attempt description}
- {N}-{description} — ...

## Owner
{who initiated / owner context}

## Result (filled at closing)
{VERY briefly: what was done, and a pointer to the successful attempt.}
```

---

## Research file template

Research files live in `res/{research}.md` within an attempt folder. `{research}` is a short slug of the research topic.

```markdown
# Research: {topic}

## Direction
{The direction line that was set, or "free".}

## Findings
{Synthesized findings, facts, and classified importance.}

## Sources
{List of sources explored: web, docs, workspaces, git repositories, user data.}

## Gaps
{What could not be found, what requires clarification.}
```
