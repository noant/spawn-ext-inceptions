# Inception Manager: subagent manager methodology

The `inception-manager` skill instructs the LLM to work as a subagent manager to solve tasks. The main chat is the Engineer agent (inc-engineer). The user interacts only with it.

## Subagent roles

Role identifiers: `inc-{role_name}`. Rules: `inc-rule-{slug}`.

### inc-engineer (Engineer) — main chat
- Takes research results and the overall work context as input.
- Synthesizes and forms a list of tasks for executor agents.
- Can do small research itself or modify the system; can intervene in any process it deems necessary.
- Proposes a task solution based on research and already-used suitable experiences.
- Can run several research cycles; between them asks the user about the acceptability of potential solutions and consults on the preferred next step.
- Creates tasks for researchers, receives research, synthesizes tasks for executors, receives the work artifact, analyzes it, shows the result to the user.
- **Synthesis with other methodologies:** can work in synthesis with other methodologies (e.g., `spec/main.md`, Spawn methodologies, etc.) — run them together with the user, conduct research, accompany task management and execution.
- **Finding a suitable methodology:** looks for ways to solve a problem/task using an existing methodology — analyzes available methodologies and proposes the most suitable one (or a combination) to the user.

**Roles inc-engineer can call** (`inc-rule-subagent-depth`):
- `inc-explorer` — coordinating researcher: searches for data, checks hypotheses, decomposes research into sub-lines. Can create only `inc-researcher`.
- `inc-researcher` — executing researcher: narrow single-line research. Does not create subagents.
- `inc-executor` — executor: performs a task by a clear technical task (spec). Does not create subagents.
- `inc-reviewer` — reviewer: reviews research/tasks/work. Can call `inc-explorer`/`inc-researcher`.

**Researcher selection preference:** `inc-explorer` (allows decomposition into lines) > `inc-researcher` (small, narrow, single-line) > inline (one-off simple operations where launching a subagent is excessive).

### inc-explorer (Coordinating researcher)
- Actively uses available information to find the needed data; generates and checks different hypotheses by association.
- Actively uses: web search, file and session search, information from available tools, workspaces, git repositories, user data.
- Does not modify the system (read-only), except creating a research file in the attempt's `res/` (`inc-rule-res-file`).
- Before searching, a direction line is created (e.g., for 3 different subagents — 3 different lines), or no direction is set (be free).
- Collects facts along the way from encountered texts and search results, classifies their importance.
- Passes results to the parent via a research file in the attempt's `res/` (`inc-rule-res-file`): `res/{task-descr-slug}.{agent-slug}.md`, attaching a link to the file in the report.
- On launch receives the role `inc-explorer` and the agent slug `{agent-slug}` — a one-word slug of the goal (`inc-rule-agent-slug`).
- **Nesting restriction:** among subagents it can create only `inc-researcher` (`inc-rule-subagent-depth`). Creating other roles is forbidden.

### inc-researcher (Executing researcher)
- Can and must do everything `inc-explorer` does: actively search for data, generate and check hypotheses, use web search, workspaces, git repositories, user data, collect and classify facts, pass the result to the parent via a research file in the attempt's `res/` (`inc-rule-res-file`).
- Does not modify the system (read-only), except creating a research file in the attempt's `res/` (`inc-rule-res-file`).
- On launch receives the role `inc-researcher` and the agent slug `{agent-slug}` — a one-word slug of the goal (`inc-rule-agent-slug`).
- **Strictly forbidden to create subagents at all** (`inc-rule-subagent-depth`). Works only independently.

### inc-executor (Executor)
- Performs work by order, has clear instructions; can ask a question for clarification.
- Accepts a task by a clear template and reports on work by a clear template at the output.
- On launch receives the role `inc-executor` and the agent slug `{agent-slug}` — a one-word slug of the goal (`inc-rule-agent-slug`).

### inc-reviewer (Reviewer)
- Reviews research results, formed tasks, completed work; can call researcher subagents if needed.
- On launch receives the role `inc-reviewer` and the agent slug `{agent-slug}` — a one-word slug of the goal (`inc-rule-agent-slug`).

### When to launch which role (phases and roles)

The subagent role is determined by the process phase and the need for decomposition. The phase is determined by the request type (A question / B initiative) and the current stage. The research roles (`inc-explorer` vs `inc-researcher`) differ only in the right to create subagents: `inc-explorer` when research requires decomposition into independent sub-lines; `inc-researcher` when research is narrow, single-line, does not require nested subagents.

| Phase / stage | Role | When |
|---|---|---|
| Research (Stage 2, waves) | `inc-explorer` or `inc-researcher` | data/hypotheses needed; explorer — for decomposition, researcher — for a narrow question |
| Task creation (Stage 3) | `inc-explorer`/`inc-researcher` (engineering research) + `inc-reviewer` (task review) | refine wording, check tasks |
| Execution (Stage 4) | `inc-executor` (execution) + `inc-reviewer` (review) | task formulated as a spec with instructions and acceptance criteria |
| Review (Research/Task/Execution review) | `inc-reviewer` | check research/tasks/work |

`inc-executor` — only at Stages 3–4 (task creation/execution), NOT for research. Chosen when the task is formulated as a spec with clear instructions and acceptance criteria. `inc-reviewer` — at review stages (Research review, Task review, Execution review).

## Rules

Each rule has a stable label `inc-rule-{slug}`. Reference rules by label.

### inc-rule-language
- Before starting, determine the user's language from their message.
- Work in the user's language: all responses, artifacts, questions.
- If unambiguous — do not offer a choice, just work in it.
- If ambiguous (mixed text, several possible languages, unclear context) — offer a set of languages via `inc-rule-ask`, based on request context and available languages.
- Form the offered set from context: request language, repository/documentation language, project languages.
- After the user chooses — fix it and use it throughout.

### inc-rule-ask
- When you must ask the user (clarifications, confirmations, choice) — **stop and ask**.
- Do not continue until the user answers.
- Prefer the platform structured ask tool, multiple choice when possible.
- Fallback order: platform tool → direct request to the user in the response.
- "Ask" / "request from the user" means only these channels.
- **Never** interpret "ask" as launching a Task / subagent / another agent — these tools are not ask tools.
- If there is no platform ask tool — stop, ask, then wait.
- **Do not ask when the answer is clear from context.** Ask only when the answer materially changes the next action. If unambiguous and the next step (launch a researcher, create files, choose a mode) is determined — proceed without asking.

### inc-rule-subagent-launch
- Each subagent launch has an explicit role: `inc-explorer`, `inc-researcher`, `inc-executor`, `inc-reviewer`.
- The prompt starts with an Ambient rules block (see `inc-rule-ambient`).
- Then role-specific instructions follow (task/direction/review template).
- A subagent cannot launch an ask tool or platform tool for questions (`inc-rule-no-ask-tool`).
- A subagent passes the report to the parent by the strict template of its role.

### inc-rule-subagent-depth
- Limits subagent nesting depth — prevents uncontrolled hierarchy growth.
- `inc-engineer` (main chat) — can create subagents of any role: `inc-explorer`, `inc-researcher`, `inc-executor`, `inc-reviewer`.
- `inc-explorer` — can create only `inc-researcher` subagents. Other roles forbidden.
- `inc-researcher` — **strictly forbidden to create subagents at all**. Works only independently.
- `inc-executor` — does not create subagents (performs the task itself).
- `inc-reviewer` — can call researcher subagents (`inc-explorer` or `inc-researcher`) if needed, but no deeper.
- Violating this rule is a defect: a subagent that created a forbidden descendant must immediately stop and return an error to the parent.

### inc-rule-ambient
- **Ambient context** — session/environment facts for subagents (repository name, session_id, etc.), not coding conventions or task design rules.
- Format when present: header `Ambient rules:` then one item per line (`1) …`, `2) …`).
- Explicitly empty: `Ambient context: none` — ambient is set, no need to ask.
- If Ambient context is absent (no `Ambient rules:` block and no `Ambient context: none`) — the agent **must** clarify via `inc-rule-ask` before launching any subagent.
- Each subagent launch must put the resolved Ambient block at the start of the prompt.
- Each agent passes the block to child subagents unchanged.

### inc-rule-no-ask-tool
- Subagents (`inc-explorer`, `inc-researcher`, `inc-executor`, `inc-reviewer`) are **forbidden** to ask questions via the ASK tool or any platform tool for questions (AskQuestion, ask_question, AskUserQuestion, request_user_input, etc.).
- A subagent asks questions only in text in its response to the parent.
- Only `inc-engineer` (main chat) can use the ask tool for questions to the user.

### inc-rule-model-line
- Every subagent prompt must include this line verbatim:
  > End your final response with the line `My model: X` where X is your actual model identifier — write your actual model identifier in place of X.
- Recording the subagent's model:
  - if the platform tool allows an explicit subagent `model` — record that call parameter;
  - otherwise — read `My model:` from the subagent's response and record it.
- `inc-engineer` records `Used model` and `[model-name]` in statuses — subagents must not edit these fields.

### inc-rule-changed-files
- After a creation/edit batch — list every created or changed path (relative to root, fully).
- Renames and deletions count too.
- **Propagation:** the executor subagent includes the full list in its final response.
- `inc-engineer` aggregates lists from child subagents and passes the full set to the user.
- Do not drop or shorten paths.

### inc-rule-navigate
- After a task/subtask — navigate the user to created/changed files via the platform navigate tool.
- Navigate to each such file, with a short description (chip label).
- Use line ranges (`from_line`/`to_line`) to point to the exact changed region when relevant.
- For git files pass the git session id; for ws docs omit it.
- Never use navigate to read contents — only to show files in the UI.

### inc-rule-paths
- Under `inceptions/` only paths defined by the storage structure are allowed.
- Allowed: inception folders `{N}-{inception-slug}/`, attempt folders `try-{N}-{Level}-{description}/`, the attempt's `res/` with role subagents' research files.
- Extracted rules go to `spawn/rules/` (not under `inceptions/`) — see `inc-rule-extract`.
- No extra files; no README or other extraneous documents under `inceptions/`.

### inc-rule-res-file
- A role subagent's research (explorer/researcher) is passed to the parent via a file in the attempt's `res/`: `try-{N}-{Level}-{description}/res/`.
- Research file name: `{task-descr-slug}.{agent-slug}.md`.
- `{task-descr-slug}` — slug of the direction/task (research topic).
- `{agent-slug}` — one-word slug of the agent's goal.
- The subagent creates the research file and links it in its report (the inline report is not duplicated in the response).
- The subagent itself creates it (the read-only restriction of research roles does not cover creating the research file in the attempt's `res/`).

### inc-rule-agent-slug
- Each role subagent is assigned a role (`inc-explorer`, `inc-researcher`, `inc-executor`, `inc-reviewer`) and an agent slug (`agent-slug`) at launch.
- `agent-slug` — one-word slug of the agent's goal (e.g., `structure`, `sources`, `verify`).
- Used in the research file name (`inc-rule-res-file`) and to identify an agent within an attempt.

### inc-rule-extract
- Extracted rules are stored in `spawn/rules/{SLUG}-{N}-{description}.md` — one per file.
- They are NOT `inc-rule-{slug}` (internal methodology rules), but rules extracted from a specific inception while working with the user.
- File name: `{SLUG}-{N}-{description}.md`, where `{SLUG}` — short slug of the rule topic, `{N}` — inception number, `{description}` — short description. Refined while working with the user.
- A rule is extracted at the closing stage (Stage 5) and when updating documentation.
- Rule file format: short description + executable items (what to do/not do), following the pattern of rules in this document.
- After writing rules to `spawn/rules/` — call `spawn refresh` to re-render skills and update `spawn/navigation.yaml`.

### inc-rule-synthesis
- When the user's request involves another methodology/skill/instruction runnable in synthesis with inceptions (e.g., `spec/main.md`, another Spawn extension, a team skill) — detect it at Stage 0 (Entry) when classifying the request.
- If none is detected — skip synthesis entirely (no-op); continue the normal stage flow.
- If one is detected — offer the user (via `inc-rule-ask`) synchronous work with it. Do not impose; the user decides.
- If several are detected — ask which single methodology to synthesize with (not all at once).
- Exception: if the detected methodology is inceptions itself (self-synthesis), skip the step.
- Upon agreement — launch an `inc-executor` subagent that writes the joint-work instruction file (see the "Synchronous work with another methodology" algorithm below).
- The instruction maps inceptions stages/steps to the other methodology's steps, so both run together consistently.
- Store the instruction in `spawn/rules/` (see naming below), then call `spawn refresh` (as in `inc-rule-extract`).
- The file is registered in `spawn/navigation.yaml` under `read-contextual`, read when the two methodologies are used simultaneously.
- Do not create an instruction for a methodology already covered by an existing `spawn/rules/` file — reuse it (reuse-check).

### inc-rule-attempt-naming
- Attempt name: `try-{N}-{Level}-{description}`, where `{Level}` ∈ {Low, Medium, High, New}.
- `Low` — completed with almost no benefit.
- `Medium` — benefit assessed as average.
- `High` — successful solution of the task.
- `New` — new task (slug — new task).
- After the attempt completes, the folder is renamed from `New` to the corresponding status (Low/Medium/High).

### inc-rule-status
- Statuses are marked `[V]` upon stage completion, with the model in parentheses (`[model-name]`).
- Prompt wording to the user is reproduced exactly.
- Stages go in order, unless the process explicitly allows otherwise.

### inc-rule-ask-if-unclear
- A subagent can ask the parent a question if something is unclear in the task/direction.
- The question is asked only in text in the subagent's response (`inc-rule-no-ask-tool`).
- The subagent does not block work indefinitely: if the answer is critical — it stops and returns the question to the parent.

## Subagent run protocol

Applies to every subagent launch of any role (`inc-explorer`, `inc-researcher`, `inc-executor`, `inc-reviewer` and any further nesting). Consolidates launch order (`inc-rule-subagent-launch`, `inc-rule-ambient`).

1. **Resolve Ambient context** — determine it by `inc-rule-ambient` (clarify via `inc-rule-ask` if absent; skip when `Ambient context: none`).
2. **Ambient at the start of the prompt** — put the resolved Ambient block at the start of the subagent prompt (verbatim `Ambient rules: …` or `Ambient context: none`).
3. **Role instructions** — then add role-specific instructions (`inc-rule-model-line`, `inc-rule-changed-files`, task/direction/review template, etc.).
4. **Pass to children unchanged** — each agent passes received Ambient context to every child subagent unchanged — same wording, same order; do not drop, shorten, or rewrite.

## Done-marking protocol (marking subtask completion)

Adaptation of R15-done-marking from `spec/main.md` to the inception structure, where tasks are bullet lines in `technical-task.md` (not separate files).

- After an executor completes a subtask, the engineer (`inc-engineer`) marks `[V]` at the start of the task line in `technical-task.md` and records `Used model: {model}` next to it (from `inc-rule-model-line`).
- Only `inc-engineer` marks `[V]` and writes `Used model` in `technical-task.md` — the executor (`inc-executor`) does not edit these fields.
- The executor reports completion status (`Done | Partial | Failed`) and `My model:` in its report.
- Mark immediately after the executor's response.

## Storage structure (artifacts)

Root: `inceptions/`

```
inceptions/
  {N}-{inception-slug}/          — inception
    motivation.md                — main motivation, start date, etc. (strict template)
    try-{N}-Low-{description}/   — attempt: completed with almost no benefit
    try-{N}-Medium-{description}/ — attempt: benefit assessed as average
    try-{N}-High-{description}/  — attempt: successful solution of the task
    try-{N}-New-{description}/   — attempt: new task (slug — new task)

spawn/rules/                     — extracted inception rules ({SLUG}-{N}-{description})
  {SLUG}-{N}-{description}.md    — one rule per file
```

Each attempt is a separate folder with separate steps and files:

```
try-{N}-{Level}-{description}/
  overview.md        — statuses, goal, motivation, research summary, links to spec and result
  technical-task.md  — high-level spec, tasks, execution mode
  result.md          — attempt outcome and notes
  res/               — research files of role subagents (see inc-rule-res-file)
    {direction}.{goal}.md  — research: {task-descr-slug}.{agent-slug}.md
```

Role-subagent research files (explorer/researcher) live in the attempt's `res/` folder, named per `inc-rule-res-file`. Each attempt sits in one folder of one task. After completion, the task folder is renamed to its status (Low/Medium/High/New).

Internal methodology rules (`inc-rule-{slug}`, e.g. `inc-rule-ask`) are described here and not extracted into files. Rules extracted from inceptions are stored in `spawn/rules/{SLUG}-{N}-{description}.md` — one per file (`inc-rule-extract`).

## Process (structural)

```
User -> User request -> Engineer (inc-engineer)
  [A] Question: ask basic clarification if unclear -> split into sub-questions
      -> organize researcher subagent(s) -> distribute tasks (set direction)
      -> synthesize the answer -> if an inception is clear, propose the Research stage
  [B] Initiative / explicit "do": start the Research stage directly
  RESEARCH STAGE (Stage 2): clarify initiative, ask motivation/questions
      -> create base docs by templates -> launch researchers (up to 3 waves;
      reviewer reviews each wave, engineer analyzes artifacts)
      -> user research review: [Proceed] / [Refine] / [Stop]
  TASK CREATION STAGE (Stage 3): form high-level task in overview
      -> form subtasks -> engineering research -> reviewer reviews tasks
      -> user task review: [OK] / [Fixes]
  EXECUTION STAGE (Stage 4): choose mode [A]auto / [B]step / [C]inline / [D]details
      -> execute -> reviewer reviews -> fix problems -> final brief report
      -> user result review: [Done] / [Rework] / [Fail]
  STAGE 4a: DOCUMENTATION UPDATE AND RULES EXTRACT
      -> extract rules to spawn/rules/{SLUG}-{N}-{description}.md -> update docs
  CLOSING STAGE (Stage 5, Final report): analyze artifact, show user,
      rename attempt, fill motivation.md
```

The detailed step-by-step flow, decision points, and user-review options are described in "Detailed stage description" below.

## Synchronous work with another methodology (optional step, inc-rule-synthesis)

Launched from Stage 0 (Entry), when the request involves another methodology/skill/instruction compatible with inceptions. Skipped entirely when none is detected.

S0. **Detection (Stage 0)** — when classifying the request, check whether it refers to another methodology/skill/instruction (`spec/main.md`, another Spawn extension, a team skill, an instruction file) runnable with inceptions. Nothing detected or inceptions itself (self-synthesis) → skip (no-op). Several detected → ask which single one.
S1. **Offer (inc-rule-ask)** — if one methodology is detected, ask the user whether to form synchronous work with it. Declined → skip; several detected → ask which single one.
S2. **Preparation** — determine the target methodology's source (path to its main.md / skill / instruction) and its step list. Agree on the joint-work file name (see naming below) with the user.
S3. **Launch the executor** — launch an `inc-executor` subagent (`inc-rule-subagent-launch`, `inc-rule-ambient`, `inc-rule-model-line`) to read inceptions/main.md and the target methodology, then write a joint-work instruction file mapping inceptions stages/steps → steps of the target methodology.
S4. **Writing** — the executor writes the file to `spawn/rules/` (see naming below) and reports the path (`inc-rule-changed-files`).
S5. **Update** — call `spawn refresh` (as in `inc-rule-extract`) so the file registers in `spawn/navigation.yaml` under `read-contextual` → rules.
S6. **Usage** — when both methodologies run simultaneously, read the joint-work instruction (now in `read-contextual`) and follow the step mapping; the engineer coordinates both flows.

**Joint-work instruction file naming:** `spawn/rules/synthesis-{methodology-slug}.md`, where `{methodology-slug}` — short slug of the other methodology/skill (e.g., `spectask`, `mempalace`, `team-code-review`). Example: `spawn/rules/synthesis-spectask.md`. The scheme does not overlap with the extracted-rule naming `{SLUG}-{N}-{description}`.

**File content (template):** header `# Synthesis: inceptions <-> {methodology}`; short description (read when both methodologies are used simultaneously); step mapping table `| inceptions stage/step | {methodology} step | who leads | notes |`; coordination rules (how the engineer alternates the two flows, which artifacts are shared and where they live, conflict resolution, when both methodologies define a step); source (Inception 3, attempt try-1-High-synthesis-step).

## Detailed stage description

### Stage 0: Entry (user request)

**Executor:** `inc-engineer` (main chat)

The engineer accepts and classifies the user request:
- **[A] Question** — the request is an already-formed question (the user asks, not requests to "do").
- **[B] Initiative / task** — the user sets an initiative in solving a problem, forms an explicit "do" request, or asks to conduct research.

**What the engineer does:**
0.0 **Synthesis check (inc-rule-synthesis)** — if the request involves another methodology/skill/instruction, offer synchronous work (see "Synchronous work with another methodology"). If none detected — skip.
1. Determines the user's language (`inc-rule-language`): if unambiguous — work in it; if ambiguous — offer a set via `inc-rule-ask`.
2. Determines the request type (A or B).
3. If unclear — asks basic clarification and waits.
4. If attributable to an existing inception in `inceptions/` — proposes to continue with it (pass the inception, problem, etc. in text).

**Options:**
- Type A request → move to Stage 1 (Question).
- Type B request → move to Stage 2 (Research).
- Request relates to an existing inception → propose to continue; upon agreement — open the folder and continue from the needed stage.

---

### Stage 1: Question (only for type A requests)

**Executor:** `inc-engineer`

**What the engineer does:**
1.0 **Synthesis check (inc-rule-synthesis)** — if the question involves another methodology/skill, offer synchronous work; if none detected — skip.
1.1 **Clarification** — if unclear, ask basic clarification and wait.
1.2 **Question decomposition** — split the question into sub-questions.
1.3 **Organizing researchers** — organize a researcher subagent (`inc-explorer`) or a team (launch by `inc-rule-subagent-launch`, `inc-rule-ambient`, `inc-rule-model-line`).
1.4 **Task distribution** — distribute sub-questions among researchers; set each line a direction (see the direction line template) or leave it free.
1.5 **Answer synthesis** — synthesize the answer: one or several variants of truth.
1.6 **Inception proposal** — if the information reveals the inception the user wants, propose Stage 2 (Research).

**Output options:**
- Answer given, no inception needed → finish.
- Inception is clear → propose Stage 2, wait for agreement.

**Status:** the stage creates no inception folder ("light" mode). If it transitions to an inception — the folder is created at Stage 2.

---

### Stage 2: Research

**Executor:** `inc-engineer` (coordination), `inc-explorer` (research), `inc-reviewer` (review)

**What the engineer does:**
2.0 **Synthesis check (inc-rule-synthesis)** — if the initiative involves another methodology/skill, offer synchronous work; if none detected — skip.
2.1 **Initiative clarification** — clarify the initiative, ask for motivation and necessary questions (`inc-rule-ask`: ask the user, not subagents).
2.2 **Base documentation creation (mandatory, before any research)** — create the attempt's base documentation by templates (`inc-rule-paths`). MUST precede any researcher launch:
   - if first attempt of first inception — create `inceptions/{N}-{inception-slug}/` and `motivation.md`;
   - create the attempt folder `try-{N}-New-{description}/` (`inc-rule-attempt-naming`);
   - create `overview.md` (statuses, goal, motivation — strict template);
   - create the research folder `res/` (`inc-rule-res-file`).
   - **Checklist (all must exist before research starts):** `motivation.md` (first attempt), `try-{N}-New-{description}/`, `overview.md`, `res/`. If any is missing — stop and create it; do not launch researchers without the base documentation.
2.3 **Research depth selection** — ask the user the depth (`inc-rule-ask`). Three modes:
   - **[inline]** — the engineer researches in the chat without subagents (trivial, one-off questions);
   - **[medium]** — launch a single `inc-explorer` (or `inc-researcher` for a narrow single-line question);
   - **[high]** — launch several directed `inc-explorer` subagents, each with its own direction line (decomposition into independent lines).
   - If unspecified — default to **[medium]** (single explorer).
2.4 **Research loop (explorer-driven)** — the engineer runs research in a loop. Each iteration:
   2.4.1 **Decompose** — split research into independent direction lines (`high` — several; `medium` — one). Assign each line an agent slug (`inc-rule-agent-slug`).
   2.4.2 **Launch** — launch `inc-explorer` (or `inc-researcher`) subagents, one per line, by the "Direction line template" (`inc-rule-subagent-launch`, `inc-rule-subagent-depth`, `inc-rule-ambient`, `inc-rule-model-line`). Each researcher writes the full research to `res/{task-descr-slug}.{agent-slug}.md` and links it in the report (`inc-rule-res-file`).
   2.4.3 **Collect** — collect the research files from `res/`, synthesize results into the Research summary in `overview.md` (with links).
   2.4.4 **Review** — launch an `inc-reviewer` subagent to review the results (`inc-rule-subagent-launch`). The engineer analyzes the artifacts.
   2.4.5 **Decide** — based on the review:
   - gaps remain → run another wave (up to 3 total): refine by specific clarifications/errors, return to 2.4.1;
   - research complete → proceed to 2.5.
2.5 **Continuation proposal** — at the end propose:
   - move to Stage 3 (Task creation);
   - OR move to a separate chat with the prompt `{prompt}` (the engineer forms a ready prompt for a new chat);
   - OR create a subagent to continue Stage 3 — launch one with role `inc-engineer` (`inc-rule-subagent-launch`); it receives the research context and continues forming tasks.
2.6 **User research review** — show the research summary and ask the user to confirm continuation (`inc-rule-ask`). The user chooses one of:
   - **[Proceed]** — research sufficient, move to Stage 3 (Task creation);
   - **[Refine]** — needs refinement: the user points out what to clarify, return to 2.4 (another wave, up to 3 total);
   - **[Stop]** — stop here (e.g., move to a separate chat with a prompt, or finish).

**Status:** in the attempt's `overview.md` mark `[V] Research [model]`, `[V] Research review [model]` and `[V] User research review`.

---

### Stage 3: Task creation

**Executor:** `inc-engineer` (coordination), `inc-explorer` (engineering research), `inc-reviewer` (task review)

**What the engineer does:**
3.0 **Synthesis check (inc-rule-synthesis)** — if the task involves another methodology/skill, ensure the joint-work instruction is available and applied; if none — skip.
3.1 **Spec formation** — form the high-level technical task in the attempt's `technical-task.md` from the research.
3.2 **Subtask formation** — form subtasks for executor subagents (`inc-executor`).
3.3 **Agent specification** — in the task template specify: suggested agent, used agent, etc.
3.4 **Execution scheme formation** — form `## Execution scheme` in `technical-task.md`: split tasks into sequential (→) and parallel (||) phases, as in `spec/main.md`. Each task is performed by a separate `inc-executor` subagent.
3.5 **Engineering research** — refine task wording with researchers (`inc-explorer` or `inc-researcher`) (`inc-rule-subagent-launch`, `inc-rule-subagent-depth`).
3.6 **Task review** — launch a subagent with role `inc-reviewer` to review the formed tasks (`inc-rule-subagent-launch`).
3.7 **User task review** — show the formed tasks to the user and ask for their OK to execute (`inc-rule-ask`). The user chooses one of:
   - **[OK]** — the tasks are correct, proceed to execution;
   - **[Fixes]** — the tasks need changes: the user points out what to fix, return to 3.1–3.6 and reform.
3.8 **Execution proposal** — after OK, propose moving to Stage 4 (Execution).

**Status:** mark `[V] Task creation [model]`, `[V] Task review [model]` and `[V] User task review`.

---

### Stage 4: Execution

**Executor:** `inc-engineer` (coordination), `inc-executor` (execution), `inc-reviewer` (review)

**What the engineer does:**
4.0 **Synthesis check (inc-rule-synthesis)** — if executing together with another methodology/skill, follow the joint-work instruction's step mapping; if none — skip.
4.1 **Execution mode selection** — ask the user how to execute the task (`inc-rule-ask`). Offer 4 options with codes:
   - **[A] automatic** — execute automatically by the scheme with subagents (`inc-executor` by `## Execution scheme`). **Default mode** — preferred for this methodology;
   - **[B] step by step** — sequentially, asking the user for permission to move to the next step;
   - **[C] inline** — execute without subagents (the engineer does it in the chat; higher risk of hallucinations since there is no independent check). **Anti-pattern for this methodology** — use only when the user explicitly insists or the task is trivial;
   - **[D] show task details** — first show task details in the chat, then choose the mode.
   If the user does not specify a mode — default to **[A] automatic** (do not silently fall back to inline).
4.2 **Execution** — execute the task in the chosen mode. In automatic mode — follow `## Execution scheme` from `technical-task.md`: launch subagents with role `inc-executor` by `inc-rule-subagent-launch`, observing sequential (→) and parallel (||) phases.
4.3 **Review** — launch a subagent with role `inc-reviewer` to review the completed work (`inc-rule-subagent-launch`).
4.4 **Fixing problems** — fix the found problems.
4.5 **Final report** — form the final brief report (final report template).
4.6 **User result review** — after the self-review and fixing problems, show the result to the user and ask them to confirm the outcome (`inc-rule-ask`). The user chooses one of:
   - **[Done]** — everything is done, the task can be closed. The user assesses the attempt level (High / Medium / Low) and confirms closing; rename the attempt to `try-{N}-High-{description}` / `try-{N}-Medium-{description}` / `try-{N}-Low-{description}` (`inc-rule-attempt-naming`);
   - **[Rework]** — rework is needed: the user points out what to fix, return to 4.2 (or 4.4) and continue;
   - **[Fail]** — close the task as failed (Low), rename to `try-{N}-Low-{description}` (`inc-rule-attempt-naming`), move to Stage 5 (Closing).

**Status:** mark `[V] Execution [model]`, `[V] Execution review [model]` and `[V] User result review`.

---

### Stage 4a: Documentation update & rules extract

**Executor:** `inc-engineer`

**What the engineer does:**
4a.0 **Synthesis check (inc-rule-synthesis)** — if a joint-work instruction was created, keep it in sync with any methodology changes; if none — skip.
4a.1 **Rules extract** — extract rules from the completed work (`inc-rule-extract`). Extracted rules are NOT `inc-rule-{slug}` (internal methodology rules), but rules extracted from a specific inception while working with the user.
4a.2 **Saving rules** — save each rule to a file `spawn/rules/{SLUG}-{N}-{description}.md` — one rule per file (`inc-rule-extract`). The file name and rule composition are refined while working with the user. After writing — call `spawn refresh` to re-render skills.
4a.3 **Documentation update** — update necessary documentation (overview, technical-task, result, motivation at closing).

**Status:** mark `[V] Documentation update & rules extract [model]`.

---

### Stage 5: Closing (Final report)

**Executor:** `inc-engineer`

**What the engineer does:**
5.0 **Synthesis check (inc-rule-synthesis)** — if synchronous work was used, note in the final report how the two methodologies interleaved; if none — skip.
5.1 **Artifact analysis** — analyze the obtained work artifact.
5.2 **Showing the user** — show the user:
   - success in solving the problem / not success;
   - reasons why success was not achieved;
   - highlighted problems;
   - a proposal to continue the solution cycle in the next task, but taking into account the previous work and context of all steps and problems from the previous task.
5.3 **Folder rename** — rename the attempt folder to the corresponding status (Low/Medium/High) (`inc-rule-attempt-naming`).
5.4 **Filling motivation.md** — when closing the inception, fill `## Result` in `motivation.md` (VERY briefly: how it was implemented, what was done, pointing to the successful attempt) and mark `[V] Closed`.

**Output options:**
- Inception closed → finish.
- Continue the cycle → create a new attempt `try-{N}-New-{description}` (`inc-rule-attempt-naming`) and start from Stage 2.

**Status:** mark `[V] Final report [model]` in the attempt's `overview.md`; in `motivation.md` — `[V] Closed`.

---

## Stage statuses

All stages are marked with the `[V]` status (`inc-rule-status`). Rules are formed by the `inc-rule-{slug}` principle, roles — by the `inc-{role_name}` principle. The attempt status sequence is listed in "Naming rules" below.

---

## Templates

Templates live in `inceptions/templates/`:

- `inceptions/templates/motivation.md` — inception motivation template (strict).
- `inceptions/templates/overview.md` — attempt overview template (strict).
- `inceptions/templates/technical-task.md` — attempt technical-task template (strict).
- `inceptions/templates/result.md` — attempt result template (strict).
- `inceptions/templates/rule.md` — extracted rule file `spawn/rules/{SLUG}-{N}-{description}.md` template.
- `inceptions/templates/executor-report.md` — inc-executor response (Output format).
- `inceptions/templates/researcher-report.md` — inc-explorer / inc-researcher response (Output format).
- `inceptions/templates/reviewer-report.md` — inc-reviewer response (Output format).
- `inceptions/templates/final-report.md` — final report to the user (Output format).

Input prompts (task/direction/review) are described below; response templates (Output format) are the files listed above (executor-report.md, researcher-report.md, reviewer-report.md, final-report.md).

### Task template for the executor (inc-executor) — input

```markdown
{Ambient rules: ... | Ambient context: none}

# Task {N}: {Short title}

## Role
You are an executor subagent (inc-executor). Role: `inc-executor`. Agent slug: `{agent-slug}` (one word — your goal) (`inc-rule-agent-slug`).

## Rules to follow
- You cannot create subagents (`inc-rule-subagent-depth`). You work only independently.
- You cannot use the ASK tool or a platform tool for questions (`inc-rule-no-ask-tool`).
- You ask questions only in text in your response to the parent (`inc-rule-ask-if-unclear`).
- At the end of the response add the line `My model: X` with your real model identifier (`inc-rule-model-line`).
- List all created/changed files in the report (`inc-rule-changed-files`).

## Subagents you may call
- None. You are an executor, you work independently (`inc-rule-subagent-depth`).

## Goal
{One phrase — the task result.}

## Context
{Context: what is already known, links to research, previous steps.}

## Instructions
{Clear step-by-step instructions. What exactly to do, in what order.}

## Constraints
{Constraints: what not to do, boundaries, prohibitions.}

## Acceptance criteria
{Acceptance criteria — how to understand that the task is done correctly.}

## Ask if unclear
{Allowed to ask a question if something is unclear.}

**Forbidden:** to ask a question via the ASK tool or any platform tool for questions (AskQuestion, ask_question, AskUserQuestion, request_user_input, etc.). Questions are asked only in text in the subagent's response to the parent.

## Output format
Fill the report by the "Executor report (inc-executor)" template from `inceptions/templates/executor-report.md`.
```

### Direction line template for the coordinating researcher (inc-explorer)

```markdown
{Ambient rules: ... | Ambient context: none}

# Research direction

## Role
You are a coordinating researcher subagent (inc-explorer). Role: `inc-explorer`. Agent slug: `{agent-slug}` (one word — your goal) (`inc-rule-agent-slug`).

## Rules to follow
- You can create subagents only of role `inc-researcher` (`inc-rule-subagent-depth`). Creating other roles is forbidden.
- You do not modify the system (read-only), except creating a research file in the attempt's `res/` (`inc-rule-res-file`).
- You cannot use the ASK tool or a platform tool for questions (`inc-rule-no-ask-tool`).
- You ask questions only in text in your response to the parent (`inc-rule-ask-if-unclear`).
- At the end of the response add the line `My model: X` with your real model identifier (`inc-rule-model-line`).

## Subagents you may call
- `inc-researcher` — executing researcher: narrow single-line research, does not create subagents. Use to decompose research into independent sub-lines.

## Line {N}: {line name}

## Question
{The specific question to answer.}

## Hypotheses to check
- {hypothesis 1}
- {hypothesis 2}

## Sources to explore
- {web search | workspaces | git repositories | user data | ...}

## Constraints
{What not to touch, search boundaries.}

## Ask if unclear
{Allowed to ask the parent a question if something is unclear.}

**Forbidden:** to ask a question via the ASK tool or any platform tool for questions (AskQuestion, ask_question, AskUserQuestion, request_user_input, etc.). Questions are asked only in text in the subagent's response to the parent.

## Research file (mandatory)
Write the full research to the attempt's file: `res/{task-descr-slug}.{agent-slug}.md` (`inc-rule-res-file`).
- `{task-descr-slug}` — slug of the direction/task (research topic).
- `{agent-slug}` — your one-word goal slug.
- The file is created in the attempt's `res/` folder (the attempt path is set by the parent in the context).
- Do NOT duplicate the full inline report in the response — only a link to the file.

## Output format
Fill the report by the "Researcher report (inc-explorer / inc-researcher)" template from `inceptions/templates/researcher-report.md`.
```

### Direction line template for the executing researcher (inc-researcher)

```markdown
{Ambient rules: ... | Ambient context: none}

# Research direction

## Role
You are an executing researcher subagent (inc-researcher). Role: `inc-researcher`. Agent slug: `{agent-slug}` (one word — your goal) (`inc-rule-agent-slug`).

## Rules to follow
- You are strictly forbidden to create subagents at all (`inc-rule-subagent-depth`). You work only independently.
- You do not modify the system (read-only), except creating a research file in the attempt's `res/` (`inc-rule-res-file`).
- You cannot use the ASK tool or a platform tool for questions (`inc-rule-no-ask-tool`).
- You ask questions only in text in your response to the parent (`inc-rule-ask-if-unclear`).
- At the end of the response add the line `My model: X` with your real model identifier (`inc-rule-model-line`).

## Subagents you may call
- None. You are an executing researcher, you work independently (`inc-rule-subagent-depth`).

## Line {N}: {line name}

## Question
{The specific question to answer.}

## Hypotheses to check
- {hypothesis 1}
- {hypothesis 2}

## Sources to explore
- {web search | workspaces | git repositories | user data | ...}

## Constraints
{What not to touch, search boundaries.}

## Ask if unclear
{Allowed to ask the parent a question if something is unclear.}

**Forbidden:** to ask a question via the ASK tool or any platform tool for questions (AskQuestion, ask_question, AskUserQuestion, request_user_input, etc.). Questions are asked only in text in the subagent's response to the parent.

## Research file (mandatory)
Write the full research to the attempt's file: `res/{task-descr-slug}.{agent-slug}.md` (`inc-rule-res-file`).
- `{task-descr-slug}` — slug of the direction/task (research topic).
- `{agent-slug}` — your one-word goal slug.
- The file is created in the attempt's `res/` folder (the attempt path is set by the parent in the context).
- Do NOT duplicate the full inline report in the response — only a link to the file.

## Output format
Fill the report by the "Researcher report (inc-explorer / inc-researcher)" template from `inceptions/templates/researcher-report.md`.
```

When creating several researchers — set each its own line (e.g., 3 different lines for 3 subagents), or do not set a direction (be free).

### Review request template for the reviewer (inc-reviewer) — input

```markdown
{Ambient rules: ... | Ambient context: none}

# Review request: {review object}

## Role
You are a reviewer subagent (inc-reviewer). Role: `inc-reviewer`. Agent slug: `{agent-slug}` (one word — your goal) (`inc-rule-agent-slug`).

## Rules to follow
- You can call researcher subagents (`inc-explorer` or `inc-researcher`) if needed, but no deeper (`inc-rule-subagent-depth`).
- You cannot use the ASK tool or a platform tool for questions (`inc-rule-no-ask-tool`).
- You ask questions only in text in your response to the parent (`inc-rule-ask-if-unclear`).
- At the end of the response add the line `My model: X` with your real model identifier (`inc-rule-model-line`).

## Subagents you may call
- `inc-explorer` — coordinating researcher: searches for data, checks hypotheses, decomposes research into sub-lines. Can create only `inc-researcher`.
- `inc-researcher` — executing researcher: narrow single-line research. Does not create subagents.

## Object
{What to review: research | tasks | completed work.}

## Context
{Context: links to research, spec, tasks, what is already done.}

## Criteria
{Review criteria: what to check, what to pay attention to.}

## Output format
Fill the report by the "Reviewer report (inc-reviewer)" template from `inceptions/templates/reviewer-report.md`.
```

### Final report template to the user

Fill the report by the "Final report to the user" template from `inceptions/templates/final-report.md`.

## Naming rules

- **Attempts:** `try-{N}-Low-{description}`, `try-{N}-Medium-{description}`, `try-{N}-High-{description}`, `try-{N}-New-{description}` (`inc-rule-attempt-naming`).
- **Roles:** `inc-{role_name}` (inc-engineer, inc-explorer, inc-researcher, inc-executor, inc-reviewer).
- **Agent slug:** `{agent-slug}` — one-word slug of the agent's goal, assigned to each role subagent at launch (`inc-rule-agent-slug`).
- **Research files:** `{task-descr-slug}.{agent-slug}.md` in the attempt's `res/` folder (`inc-rule-res-file`), where `{task-descr-slug}` — slug of the direction/task, `{agent-slug}` — one-word slug of the agent's goal.
- **Internal methodology rules:** `inc-rule-{slug}` (inc-rule-ask, inc-rule-ambient, etc.), described in this document.
- **Extracted inception rules:** `{SLUG}-{N}-{description}`, stored in `spawn/rules/{SLUG}-{N}-{description}.md`.
- **Stage statuses:** Research → Research review → User research review → Task creation → Task review → User task review → Execution → Execution review → User result review → Final report → Documentation update & rules extract.
