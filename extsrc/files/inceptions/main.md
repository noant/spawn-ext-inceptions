# Inception Manager: subagent manager methodology

The `inception-manager` skill instructs the LLM to work as a subagent manager to solve tasks. The main chat is the Engineer agent (inc-engineer). The user interacts only with it.

## Subagent roles

Role identifiers: `inc-{role_name}`. Rules: `inc-rule-{slug}`.

### inc-engineer (Engineer) — main chat
- Takes research results and the overall work context as input.
- Synthesizes and forms a list of tasks for executor agents.
- Can conduct small research itself or modify the system.
- Can intervene in any process it deems necessary to solve the task.
- Proposes a way to solve the task based on research and already-used suitable experiences.
- Can run several research cycles; between them asks the user about the acceptability of potential solutions and consults on the preferred next step.
- Creates tasks for researchers, receives research, synthesizes tasks for executors, receives the work artifact, analyzes it, shows the result to the user.
- **Synthesis with other methodologies:** can work in synthesis with other methodologies (e.g., `spec/main.md`, Spawn methodologies, etc.) — run them together with the user, conduct research, accompany task management, and execution.
- **Finding a suitable methodology:** looks for ways to solve a problem/task using an existing methodology — analyzes available methodologies and proposes the most suitable one (or a combination) to the user.

**Roles inc-engineer can call** (`inc-rule-subagent-depth`):
- `inc-explorer` — coordinating researcher: searches for data, checks hypotheses, decomposes research into sub-lines. Can create only `inc-researcher`.
- `inc-researcher` — executing researcher: narrow single-line research. Does not create subagents.
- `inc-executor` — executor: performs a task by a clear technical task (spec). Does not create subagents.
- `inc-reviewer` — reviewer: reviews research/tasks/work. Can call `inc-explorer`/`inc-researcher`.

**Researcher selection preference:**
1. `inc-explorer` — preferred choice for research (allows decomposition into lines).
2. `inc-researcher` — for small, narrow, single-line research.
3. inline (no subagent) — for one-off simple operations where launching a subagent is excessive.

### inc-explorer (Coordinating researcher)
- Actively uses available information to find the needed data.
- Generates and checks different hypotheses by association.
- Actively uses: web search, file and session search, information from available tools.
- Explores all possible sources: available workspaces, access to spaces through tools, git repositories, web search, user data.
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
- Performs work by order, has clear instructions.
- Can ask a question for clarification.
- Accepts a task by a clear template.
- Reports on work by a clear template at the output.
- On launch receives the role `inc-executor` and the agent slug `{agent-slug}` — a one-word slug of the goal (`inc-rule-agent-slug`).

### inc-reviewer (Reviewer)
- Reviews research results, formed tasks, completed work.
- Can call researcher subagents if needed.
- On launch receives the role `inc-reviewer` and the agent slug `{agent-slug}` — a one-word slug of the goal (`inc-rule-agent-slug`).

## Choosing a subagent role (when to launch which)

The subagent role is determined by the process phase and the need for decomposition. The phase is determined by the request type (A question / B initiative) and the current stage.

### inc-explorer vs inc-researcher (research roles)

Both roles search for data, check hypotheses, read-only. The difference is only in the right to create subagents:

- **inc-explorer** (coordinating researcher) — when research requires decomposition into independent sub-lines that need to be launched in parallel/sequentially by subagents. Can create only `inc-researcher`.
- **inc-researcher** (executing researcher) — when research is narrow, single-line, does not require nested subagents. Works alone.

**Selection rule:** if answering the question does not require child researchers — use `inc-researcher` (cheaper, less nesting). If decomposition into several lines is needed — use `inc-explorer`.

### Phases and roles

| Phase / stage | Role | When |
|---|---|---|
| Research (Stage 2, waves) | `inc-explorer` or `inc-researcher` | data/hypotheses needed; explorer — for decomposition, researcher — for a narrow question |
| Task creation (Stage 3) | `inc-explorer`/`inc-researcher` (engineering research) + `inc-reviewer` (task review) | refine wording, check tasks |
| Execution (Stage 4) | `inc-executor` (execution) + `inc-reviewer` (review) | task formulated as a spec with instructions and acceptance criteria |
| Review (Research/Task/Execution review) | `inc-reviewer` | check research/tasks/work |

**inc-executor** — only at Stages 3–4 (task creation/execution), NOT for research. Chosen when the task is formulated as a spec with clear instructions and acceptance criteria.

**inc-reviewer** — at review stages (Research review, Task review, Execution review).

## Rules

Each rule has a stable label `inc-rule-{slug}`. Reference rules by label.

### inc-rule-language
- Before starting work, determine the user's language from their message (from the request text).
- Work in the user's language: all responses, artifacts, questions — in the user's language.
- If the language is unambiguous — do not offer a language choice, just work in it.
- If the language is ambiguous (mixed text, several possible languages, unclear context) — offer the user a set of languages to work in (via `inc-rule-ask`), based on the request context and available languages.
- Form the set of offered languages based on context: the request language, the repository/documentation language, the languages in which the project is conducted.
- After the user chooses a language — fix it and use it throughout the work.

### inc-rule-ask
- When you need to ask the user (clarifications, confirmations, choice) — **stop and ask the user**.
- Do not continue work until the user answers.
- Prefer the platform structured ask tool, multiple choice when possible.
- Fallback order: platform tool → direct request to the user in the response.
- "Ask" / "request from the user" means only these channels.
- **Never** interpret "ask" as launching a Task / subagent / another agent — these tools are not ask tools.
- If there is no platform ask tool — stop, ask the user, then wait.

### inc-rule-subagent-launch
- Each subagent launch is performed with an explicit role: `inc-explorer`, `inc-researcher`, `inc-executor`, `inc-reviewer`.
- The subagent prompt starts with an Ambient rules block (see `inc-rule-ambient`).
- Then role-specific instructions follow (task/direction/review template).
- A subagent cannot launch an ask tool or platform tool for questions (`inc-rule-no-ask-tool`).
- A subagent passes the report to the parent by the strict template of its role.

### inc-rule-subagent-depth
- Limits subagent nesting depth — prevents uncontrolled hierarchy growth.
- `inc-engineer` (main chat) — can create subagents of any role: `inc-explorer`, `inc-researcher`, `inc-executor`, `inc-reviewer`.
- `inc-explorer` — can create subagents only of role `inc-researcher`. Creating other roles is forbidden.
- `inc-researcher` — **strictly forbidden to create subagents at all**. Works only independently.
- `inc-executor` — does not create subagents (performs the task itself).
- `inc-reviewer` — can call researcher subagents (`inc-explorer` or `inc-researcher`) if needed, but no deeper.
- Violating this rule is a defect: a subagent that created a forbidden descendant must immediately stop and return an error to the parent.

### inc-rule-ambient
- **Ambient context** — facts about the session/environment for subagents (repository name, session_id, etc.), not about coding conventions or task design rules.
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
- Each subagent prompt must include the line verbatim:
  > End your final response with the line `My model: X` where X is your actual model identifier — write your actual model identifier in place of X.
- Recording the model used by the subagent:
  - if the platform tool allows passing an explicit subagent `model` — record that call parameter;
  - if there is no model selection parameter — read `My model:` from the subagent's response and record it.
- `inc-engineer` records `Used model` and `[model-name]` in parentheses of statuses — subagents must not edit these fields.

### inc-rule-changed-files
- After completing a creation/edit batch — list every created or changed path (relative to root, fully, without omissions).
- Renames and deletions also count.
- **Propagation:** the executor subagent includes the full list in its final response.
- `inc-engineer` aggregates lists from child subagents and passes the full set to the user.
- Do not drop or shorten paths.

### inc-rule-paths
- Under `inceptions/` only paths defined by the storage structure are allowed.
- Allowed: inception folders `{N}-{inception-slug}/`, attempt folders `try-{N}-{Level}-{description}/`, the attempt's research folder `res/` with research files of role subagents.
- Extracted rules are stored in `spawn/rules/` (not under `inceptions/`) — see `inc-rule-extract`.
- No extra files.
- Do not create README or other extraneous documents under `inceptions/`.

### inc-rule-res-file
- A role subagent's research (explorer/researcher) is passed to the parent via a file in the attempt's `res/` folder: `try-{N}-{Level}-{description}/res/`.
- Research file name: `{task-descr-slug}.{agent-slug}.md`.
- `{task-descr-slug}` — slug of the direction/task (research topic).
- `{agent-slug}` — one-word slug of the agent's goal (goal reduced to one word).
- The subagent creates the research file and attaches a link to it in its report (the full inline report is not duplicated in the response).
- The research file is created by the subagent itself (the read-only restriction of research roles does not extend to creating the research file in the attempt's `res/`).

### inc-rule-agent-slug
- Each role subagent is assigned a role (`inc-explorer`, `inc-researcher`, `inc-executor`, `inc-reviewer`) and an agent slug (`agent-slug`) at launch.
- `agent-slug` — one-word slug of the agent's goal: the agent's goal/direction reduced to one word (e.g., `structure`, `sources`, `verify`).
- The agent slug is used in the research file name (`inc-rule-res-file`) and to identify a specific agent within an attempt.

### inc-rule-extract
- Extracted rules are stored in `spawn/rules/{SLUG}-{N}-{description}.md` — one rule per file.
- Extracted rules are NOT `inc-rule-{slug}` (internal methodology rules), but rules extracted from a specific inception while working with the user.
- File name: `{SLUG}-{N}-{description}.md`, where `{SLUG}` — short slug of the rule topic, `{N}` — inception number, `{description}` — short description. Refined while working with the user.
- A rule is extracted at the closing stage (Stage 5) and when updating documentation.
- Rule file format: short description + executable items (what to do/not do), following the pattern of rules in this document.
- After writing rules to `spawn/rules/` — call `spawn refresh` to re-render skills and update `spawn/navigation.yaml`.

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

Applies to every subagent launch of any role (`inc-explorer`, `inc-researcher`, `inc-executor`, `inc-reviewer` and any further nesting). Consolidates the launch order (`inc-rule-subagent-launch`, `inc-rule-ambient`).

1. **Resolve Ambient context** — determine the Ambient context by `inc-rule-ambient` (clarify via `inc-rule-ask` if absent; do not ask when `Ambient context: none`).
2. **Ambient at the start of the prompt** — put the resolved Ambient block at the very start of the subagent prompt (verbatim `Ambient rules: …` or `Ambient context: none`).
3. **Role instructions** — then add role-specific instructions (`inc-rule-model-line`, `inc-rule-changed-files`, task/direction/review template, etc.).
4. **Pass to children unchanged** — each agent that received Ambient context passes it to every child subagent unchanged — same wording, same order; do not drop, shorten, or rewrite.

## Done-marking protocol (marking subtask completion)

Adaptation of R15-done-marking from `spec/main.md` to the inception structure, where tasks are bullet lines in `technical-task.md` (not separate files).

- After an executor completes a subtask, the engineer (`inc-engineer`) marks `[V]` at the start of the task line in `technical-task.md` and records `Used model: {model}` next to it (from `inc-rule-model-line`).
- Only `inc-engineer` marks `[V]` and writes `Used model` in `technical-task.md` — the executor (`inc-executor`) does not edit these fields.
- The executor reports completion status (`Done | Partial | Failed`) and `My model:` in its report.
- Make the mark immediately after the executor's response, do not postpone.

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

Research files of role subagents (explorer/researcher) are stored in the attempt's `res/` folder. The file name is formed by `inc-rule-res-file`.

Each attempt is stored within a single folder of a single task. After completion, the task folder is renamed to the corresponding status (Low/Medium/High/New).

Internal methodology rules (`inc-rule-{slug}`, e.g. `inc-rule-ask`) are described in this document and are not extracted into separate files. Rules extracted from inceptions are stored in `spawn/rules/{SLUG}-{N}-{description}.md` — one rule per file (`inc-rule-extract`).

## Process (structural)

```
User -> User request
  |
  v
Engineer (inc-engineer)
  |
  +-- [A] Is the request an already-formed question?
  |     |
  |     +-- ask basic clarification if something is unclear, wait for the answer
  |     +-- split the question into several questions
  |     +-- organize a researcher subagent or a team of subagents
  |     +-- distribute tasks (set the direction)
  |     +-- synthesize the answer from the received information (one or several variants of truth)
  |     +-- if it is clear what inception the user wants -> propose the Research stage
  |
  +-- [B] The user immediately starts the research stage
        (initiative in solving a problem, explicit "do" request, request to conduct research)
        |
        v
  RESEARCH STAGE
        - clarify the initiative, ask for motivation, ask questions
        - create necessary files by templates
        - launch researcher agents
        - form the attempt documentation folder (if first attempt of first inception — also create inception folders)
        - create the research results file
        - there can be several research cycles (up to 3 waves):
            wave 1: researchers form base artifacts
                     -> reviewer reviews the results
                     -> engineer analyzes the artifacts
            wave 2..3: researchers refine by specific clarifications/errors
        - at the end propose: move to the task creation stage
          OR move to a separate chat with the prompt: {prompt}
        |
        v
  TASK CREATION STAGE
        - engineer forms the high-level technical task in the attempt's overview
        - forms subtasks for subagents
        - specifies in the template: suggested subagent, used subagent, etc.
        - conducts engineering research with researchers to form tasks
        - reviewer reviews the tasks
        - waits for the user's OK to execute
        - after OK -> propose moving to the execution stage
        |
        v
  EXECUTION STAGE
        - engineer asks how to execute the task (4 options):
            * [A] automatic — automatically by the scheme with subagents
            * [B] step by step — sequentially with confirmation of each step
            * [C] inline — without subagents, the engineer does it in the chat (risk of hallucinations)
            * [D] show task details — first show details, then choose the mode
        - execution
        - reviewer reviews the completed work
        - fixing problems
        - forming the final brief report
        |
        v
  STAGE 4a: DOCUMENTATION UPDATE AND RULES EXTRACT
        - extract rules from the completed work (while working with the user)
        - save rules to spawn/rules/{SLUG}-{N}-{description}.md
        - update necessary documentation (overview, technical-task, result, motivation)
        |
        v
  CLOSING STAGE (Final report)
        - analyze the artifact, show the user, rename the attempt, fill motivation.md
```

## Detailed stage description

### Stage 0: Entry (user request)

**Executor:** `inc-engineer` (main chat)

The engineer accepts the user request and classifies it:

- **[A] Question** — the request is an already-formed question (the user asks, not requests to "do").
- **[B] Initiative / task** — the user sets an initiative in solving a problem, forms an explicit "do" request, or asks to conduct research.

**What the engineer does:**

1. Determines the user's language (`inc-rule-language`): if unambiguous — work in it; if ambiguous — offer a set of languages via `inc-rule-ask`.
2. Determines the request type (A or B).
3. If something is unclear — asks the user basic clarification and waits for the answer.
4. If the request can be attributed to an existing inception in `inceptions/` — proposes to continue working with it (pass the inception, problem, etc. in text).

**Options:**

- Type A request → move to Stage 1 (Question).
- Type B request → move to Stage 2 (Research).
- Request relates to an existing inception → propose to continue, upon agreement — open the corresponding folder and continue from the needed stage.

---

### Stage 1: Question (only for type A requests)

**Executor:** `inc-engineer`

**What the engineer does:**

1.1 **Clarification** — if something is unclear, ask the user basic clarification and wait for the answer.

1.2 **Question decomposition** — split the question into several sub-questions.

1.3 **Organizing researchers** — organize a researcher subagent (`inc-explorer`) or a team of subagents (launch by `inc-rule-subagent-launch`, `inc-rule-ambient`, `inc-rule-model-line`).

1.4 **Task distribution** — distribute sub-questions among researchers, set each line a direction (see the direction line template) or do not set it (be free).

1.5 **Answer synthesis** — synthesize the answer from the received information: one or several variants of truth.

1.6 **Inception proposal** — if it is clear from the received information what inception the user wants to make, propose starting Stage 2 (Research).

**Output options:**

- Answer given, no inception needed → finish.
- Inception is clear → propose Stage 2, wait for the user's agreement.

**Status:** the stage does not create an inception folder (this is a "light" mode). If it transitions to an inception — the folder is created at Stage 2.

---

### Stage 2: Research

**Executor:** `inc-engineer` (coordination), `inc-explorer` (research), `inc-reviewer` (review)

**What the engineer does:**

2.1 **Initiative clarification** — clarify the user's initiative, ask for motivation, ask necessary questions (`inc-rule-ask`: ask the user, not subagents).

2.2 **File creation** — create necessary files by templates (`inc-rule-paths`):
   - if first attempt of first inception — create the folder `inceptions/{N}-{inception-slug}/` and `motivation.md`;
   - create the attempt folder `try-{N}-New-{description}/`, `overview.md` (`inc-rule-attempt-naming`) and the research folder `res/` (`inc-rule-res-file`).

2.3 **Launching researchers** — launch researcher agents (`inc-explorer` or `inc-researcher`), set directions and agent slugs (`inc-rule-agent-slug`). Each researcher writes the full research to the attempt's file `res/{task-descr-slug}.{agent-slug}.md` and attaches a link to the file in the report (`inc-rule-res-file`). Launch by `inc-rule-subagent-launch`, `inc-rule-subagent-depth`, `inc-rule-ambient`, `inc-rule-model-line`.

2.4 **Documentation formation** — form the attempt documentation folder, create the research results file (Research summary in `overview.md`) with links to research files in `res/`.

2.5 **Research waves** — there can be several research cycles, up to 3 waves:
   - **Wave 1:** researchers form base artifacts → launch a subagent with role `inc-reviewer` to review the results (`inc-rule-subagent-launch`) → engineer analyzes the artifacts.
   - **Wave 2..3:** researchers refine by specific clarifications or errors found by the reviewer/engineer.

2.6 **Continuation proposal** — at the end propose:
   - move to Stage 3 (Task creation);
   - OR move to a separate chat with the prompt `{prompt}` (the engineer forms a ready prompt for a new chat);
   - OR create a subagent to continue working on Stage 3 (Task creation) — launch a subagent with role `inc-engineer` (`inc-rule-subagent-launch`), the subagent receives the research context and continues forming tasks.

**Output options:**

- Move to Stage 3 (Task creation).
- Move to a separate chat with a prompt.
- Create a subagent to continue working on Stage 3.
- Launch another research wave (if gaps remain, up to 3 waves).

**Status:** in the attempt's `overview.md` mark `[V] Research [model]` and `[V] Research review [model]`.

---

### Stage 3: Task creation

**Executor:** `inc-engineer` (coordination), `inc-explorer` (engineering research), `inc-reviewer` (task review)

**What the engineer does:**

3.1 **Spec formation** — based on the research, form the high-level technical task in the attempt's `technical-task.md`.

3.2 **Subtask formation** — form subtasks for executor subagents (`inc-executor`).

3.3 **Agent specification** — in the task template specify: suggested agent, used agent, etc.

3.4 **Execution scheme formation** — form `## Execution scheme` in `technical-task.md`: split tasks into sequential (→) and parallel (||) phases, as in `spec/main.md`. Each task is performed by a separate `inc-executor` subagent.

3.5 **Engineering research** — conduct engineering research with researchers (`inc-explorer` or `inc-researcher`) to refine task wording (launch by `inc-rule-subagent-launch`, `inc-rule-subagent-depth`).

3.6 **Task review** — launch a subagent with role `inc-reviewer` to review the formed tasks (`inc-rule-subagent-launch`).

3.7 **Waiting for OK** — wait for the user's OK to execute (`inc-rule-ask`).

3.8 **Execution proposal** — after OK, propose moving to Stage 4 (Execution).

**Output options:**

- OK received → move to Stage 4.
- Fixes needed → return to 3.1–3.6, reform tasks.

**Status:** mark `[V] Task creation [model]` and `[V] Task review [model]`.

---

### Stage 4: Execution

**Executor:** `inc-engineer` (coordination), `inc-executor` (execution), `inc-reviewer` (review)

**What the engineer does:**

4.1 **Execution mode selection** — ask the user how to execute the task (`inc-rule-ask`). Offer 4 options with codes:
   - **[A] automatic** — execute automatically by the scheme with subagents (`inc-executor` by `## Execution scheme`);
   - **[B] step by step** — sequentially, asking the user for permission to move to the next step;
   - **[C] inline** — execute without subagents (the engineer does it in the chat; higher risk of hallucinations since there is no independent check);
   - **[D] show task details** — first show task details in the chat, then choose the mode.

4.2 **Execution** — execute the task in the chosen mode. In automatic mode — follow `## Execution scheme` from `technical-task.md`: launch subagents with role `inc-executor` by `inc-rule-subagent-launch`, observing sequential (→) and parallel (||) phases.

4.3 **Review** — launch a subagent with role `inc-reviewer` to review the completed work (`inc-rule-subagent-launch`).

4.4 **Fixing problems** — fix the found problems.

4.5 **Final report** — form the final brief report (final report template).

**Output options:**

- Success → move to Stage 5 (Closing), rename the attempt to `try-{N}-High-{description}` (`inc-rule-attempt-naming`).
- Partial success → rename to `try-{N}-Medium-{description}` (`inc-rule-attempt-naming`).
- Failure → rename to `try-{N}-Low-{description}` (`inc-rule-attempt-naming`).

**Status:** mark `[V] Execution [model]` and `[V] Execution review [model]`.

---

### Stage 4a: Documentation update & rules extract

**Executor:** `inc-engineer`

**What the engineer does:**

4a.1 **Rules extract** — extract rules from the completed work (`inc-rule-extract`). Extracted rules are NOT `inc-rule-{slug}` (internal methodology rules), but rules extracted from a specific inception while working with the user.

4a.2 **Saving rules** — save each rule to a file `spawn/rules/{SLUG}-{N}-{description}.md` — one rule per file (`inc-rule-extract`). The file name and rule composition are refined while working with the user. After writing — call `spawn refresh` to re-render skills.

4a.3 **Documentation update** — update necessary documentation (overview, technical-task, result, motivation at closing).

**Status:** mark `[V] Documentation update & rules extract [model]`.

---

### Stage 5: Closing (Final report)

**Executor:** `inc-engineer`

**What the engineer does:**

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

## Cycle result (showing the user)

After receiving the work artifact, the engineer analyzes it and shows the user:
- success in solving the problem / not success;
- reasons why success was not achieved;
- highlighted problems;
- a proposal to continue the solution cycle in the next task, but taking into account the previous work and context of all steps and problems from the previous task.

## Stage statuses

All stages are marked with the `[V]` status (`inc-rule-status`). Rules are formed by the `inc-rule-{slug}` principle, roles — by the `inc-{role_name}` principle.

Attempt status sequence:
Research → Research review → Task creation → Task review → Execution → Execution review → Final report → Documentation update & rules extract.

---

## Templates

Artifact templates are extracted into separate files under `inceptions/templates/`:

- `inceptions/templates/motivation.md` — template for the inception's `motivation.md` (strict).
- `inceptions/templates/overview.md` — template for the attempt's `overview.md` (strict).
- `inceptions/templates/technical-task.md` — template for the attempt's `technical-task.md` (strict).
- `inceptions/templates/result.md` — template for the attempt's `result.md` (strict).
- `inceptions/templates/rule.md` — template for the extracted rule file `spawn/rules/{SLUG}-{N}-{description}.md`.
- `inceptions/templates/agent-responses.md` — response templates (Output format) of subagents: executor, explorer, researcher, reviewer, final report.

Task/direction/review templates for subagents (input prompts) are described below in this document. Response templates (Output format) are extracted into `inceptions/templates/agent-responses.md`.

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
Fill the report by the "Executor report (inc-executor)" template from `inceptions/templates/agent-responses.md`.
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
Fill the report by the "Researcher report (inc-explorer / inc-researcher)" template from `inceptions/templates/agent-responses.md`.
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
Fill the report by the "Researcher report (inc-explorer / inc-researcher)" template from `inceptions/templates/agent-responses.md`.
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
Fill the report by the "Reviewer report (inc-reviewer)" template from `inceptions/templates/agent-responses.md`.
```

### Final report template to the user

Fill the report by the "Final report to the user" template from `inceptions/templates/agent-responses.md`.

## Naming rules

- **Attempts:** `try-{N}-Low-{description}`, `try-{N}-Medium-{description}`, `try-{N}-High-{description}`, `try-{N}-New-{description}`.
- **Roles:** `inc-{role_name}` (inc-engineer, inc-explorer, inc-researcher, inc-executor, inc-reviewer).
- **Agent slug:** `{agent-slug}` — one-word slug of the agent's goal, assigned to each role subagent at launch (`inc-rule-agent-slug`).
- **Research files:** `{task-descr-slug}.{agent-slug}.md` in the attempt's `res/` folder (`inc-rule-res-file`), where `{task-descr-slug}` — slug of the direction/task, `{agent-slug}` — one-word slug of the agent's goal.
- **Internal methodology rules:** `inc-rule-{slug}` (inc-rule-ask, inc-rule-ambient, etc.), described in this document.
- **Extracted inception rules:** `{SLUG}-{N}-{description}`, stored in `spawn/rules/{SLUG}-{N}-{description}.md`.
- **Stage statuses:** Research → Research review → Task creation → Task review → Execution → Execution review → Final report → Documentation update & rules extract.
