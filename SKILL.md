---
name: token-pilot
description: Inspect the models, reasoning efforts, and Fast scope actually available in the current Codex runtime; decide whether a task benefits from subagents; and present a task-scoped execution configuration for user confirmation before spawning. Use when a user asks to optimize, route, configure, or compare model usage for a task or subagent plan.
---

# TokenPilot

Configure execution; do not silently execute it.

## Non-negotiable contract

**Single-configuration gate:** if discovery confirms only one model/effort configuration, propose exactly one agent. Do not decompose. Ignore hypothetical file size or possible latency benefits. Multiple agents are allowed only when the user explicitly asks for parallel agents, measured inputs exceed one context window, or separate execution environments are technically required.

For a new task, follow this boundary:

1. Discover the capabilities exposed by the current runtime.
2. Analyze whether the task should be split at all.
3. Recommend `model`, `reasoning_effort`, and `fast` for each proposed unit of work.
4. Show the proposed execution structure and wait for explicit user confirmation or edits.
5. Only after confirmation, spawn or dispatch work using the confirmed settings.

Before confirmation, do not spawn execution agents, edit task files, switch Fast, or modify persistent configuration. Read-only inspection needed to form the proposal is allowed.

The routing plan is temporary and applies only to the current task. Never write it to global or workspace configuration unless the user separately asks for that change.

## Discover current capabilities

Do not maintain or infer a static model catalog. Determine what is callable now, in this order:

1. Model and reasoning-effort enums exposed by the current subagent or task-creation tool.
2. The current client/runtime model list.
3. Workspace policy restrictions.
4. The current agent's model and settings as a minimum known capability.

Treat official documentation as descriptive, not proof that a model is enabled for this account, host, or workspace. Never invent model IDs or unsupported reasoning values.

Also inspect whether Fast is available and its real scope:

- If the spawn interface exposes per-task Fast, it may be recommended per subtask.
- If Fast is session-wide, label that scope and explain which proposed work it would affect.
- If Fast cannot be safely switched and restored, recommend a manual client action instead of claiming it will be applied.
- Distinguish Fast mode from a fast model. Fast changes service speed/usage for a supported model; it is not a model or reasoning effort.

If discovery is incomplete, state exactly what is confirmed and exclude unknown capabilities from the recommendation. For example: `I can currently confirm only these callable models: ...; unobserved models are not included.`

## Decide whether to decompose

Recommend a single agent when coordination overhead would outweigh the work. Otherwise identify:

- independently useful subtasks and their dependencies;
- which units can run in parallel;
- file or state conflicts that require sequencing or ownership boundaries;
- complexity, error cost, and context size;
- whether independent verification materially reduces risk.

Do not manufacture subtasks merely to exercise the skill. The number of proposed agents should follow the work, not a fixed template.

### Delegation must reduce cost or risk

Do not split a task merely to assign the same expensive configuration several times. A same-model delegation duplicates instructions, workspace context, tool output, and handoff tokens.

- Default to one agent for ordinary implementation plus deterministic tests.
- If a subtask is simple enough for a confirmed lighter model, route it to that lighter model.
- If every proposed unit needs the baseline model, keep one agent unless isolation, parallelism, or independent review provides a concrete high-value benefit.
- When task size is unknown, do not assume parallelism will pay off. Same-model parallel reading or summarization is forbidden unless the inputs are confirmed independently large, the user prioritizes elapsed time, or a context limit makes one-agent processing unsafe.
- Ordinary code review is not automatically high risk. A lighter model can review bounded diffs, run linters/tests, check requirements, inspect responsive behavior, and report reproducible defects.
- Prefer root-run deterministic checks over a separate review agent when commands or browser assertions can establish the result directly.
- In the confirmation sheet, justify every same-model child explicitly. If the justification is only “independent review,” omit it unless failure cost is material.
- “Lower elapsed time” by itself is not enough to justify several baseline-model agents when input size and urgency are unspecified.
- If only one model/configuration is confirmed, use exactly one agent. The only exceptions are: the user explicitly requests parallel agents, measured inputs exceed one context window, or separate environments are technically required. Words such as “possibly,” “potentially,” or “may be large” never satisfy this exception.

### Minimize the coordinator tax

The baseline/root model is a control plane, not a mandatory implementation gateway. After confirmation, dispatch the chosen lower-tier worker directly. Do not ask the baseline model to rewrite the specification, create a second entry prompt, pre-read every implementation file, or restate work that the worker can obtain from named artifacts.

- When the runtime can start a fresh task or process directly on the selected worker model, prefer that direct launch over spawning the worker beneath the baseline-model session. The confirmed routing sheet plus task artifact is the handoff; the baseline session does not need to remain an active parent that waits and summarizes.
- If the available interface only supports children of the current session, include that unavoidable coordinator cost in the proposal and compare it with direct single-agent completion before recommending delegation.
- Keep the coordinator to capability discovery, routing, conflict control, and final arbitration that genuinely needs it.
- Prefer a direct worker over `root → planner → implementer` chains.
- Do not create a child merely to transfer the whole conversation. If the task cannot be scoped without nearly complete conversation inheritance, keep it with the current agent unless the cheaper model's expected savings clearly exceed the transfer cost.
- Estimate routing overhead before proposing delegation. For a small or medium task, recommend no delegation when coordinator plus handoff is unlikely to be cheaper than direct completion.

### Use artifact-driven, minimum-context handoffs

Pass the smallest **semantically complete** context, not merely the shortest prompt. Compression is valid only when it preserves every requirement that could change the worker's implementation or the acceptance decision. The handoff should normally contain:

1. the exact bounded objective, its role in the larger outcome, and acceptance checks;
2. explicit file paths, relevant symbols, line ranges, or a short task document;
3. allowed write paths and ownership boundaries;
4. commands to reproduce, test, or inspect the result;
5. interaction states, data invariants, visual/behavioral quality bars, and forbidden shortcuts relevant to that unit;
6. reference artifacts such as the current implementation, screenshots, recordings, schemas, or expected outputs when parity matters;
7. only the non-obvious constraints that are not already present in repository instructions.

Let the worker read source files and task documents directly. Do not paste large files, rollout history, screenshots, logs, or prior analysis when a path and a targeted search instruction are sufficient. However, never replace a concrete experiential requirement with a vague label such as “polished,” “interactive,” or “match the original.” Preserve observable state transitions and examples. Use limited or no inherited turns when the dispatch interface supports it. When several workers need shared facts, write one compact handoff artifact rather than repeating the facts in every prompt.

Before dispatch, run a semantic-loss check: compare the handoff against the user's request and list every must-have behavior. If any must-have has no destination worker, reference artifact, or acceptance check, the split is invalid. Fix the handoff or keep that unit with a model that has the necessary context. Context reduction is never allowed to reduce the requested quality bar.

### Split large work without losing the whole

Keep decomposition for genuinely large work, but split by cohesive outcomes rather than isolated files or generic roles. Each worker must know both its local boundary and the system-level contract it can affect.

- A UI interaction unit receives the trigger, all relevant states, visible feedback, keyboard/touch behavior, and a reference for expected appearance.
- A data unit receives the schema, ambiguous-format policy, invariants, representative edge cases, and exact assertions—not only a row-count target.
- An animation unit receives the required coordinated motions, camera/framing constraints, control effects, and recorded or frame-based acceptance evidence.
- An integration unit owns cross-component wiring and runs the end-to-end acceptance checks after parallel units finish.

Prefer one shared task-contract artifact plus small per-worker delta briefs. Workers read the shared contract only where relevant; delta briefs must not silently weaken it. Parallel writers must have disjoint ownership, and integration follows implementation rather than racing it.

Workers should report a compact delta: changed files, checks run, failures, and unresolved decisions. They should not echo source files or reconstruct the full project narrative.

## Choose the minimum sufficient configuration

### Baseline ceiling

Treat the user's or root task's starting configuration as a hard capability ceiling unless the user explicitly authorizes an upgrade.

- A child or delegated task must never use a model stronger than the baseline model.
- Its reasoning effort must never exceed the baseline reasoning effort.
- For example, with baseline `gpt-5.6-sol / low`, every unit must use `gpt-5.6-sol / low` or a demonstrably lighter callable model at `low`; never `terra/medium`, `sol/medium`, or `astra/*`.
- Fast must remain at the baseline state unless the user explicitly approves changing it and the runtime supports the stated scope.
- If the task cannot be completed reliably under the ceiling, do not silently upgrade. Present the limitation and ask whether the user wants to raise the ceiling, simplify the scope, or proceed within it.

Model ordering must come from the current runtime's descriptions or an explicit user-provided ordering. If relative strength is not confirmed, keep the baseline model rather than guessing that another model is lighter.

### Route routine work downward

Lower-tier models are capable of substantial bounded work. When the runtime confirms their availability and relative tier, prefer them for:

- straightforward HTML/CSS/JavaScript or localized code changes;
- implementing a well-specified function, component, test, or adapter;
- code review against an explicit checklist;
- linting, test execution, browser verification, data validation, and defect reproduction;
- repository search, inventory, formatting, summarization, and documentation;
- mechanical fixes whose expected output and acceptance checks are clear.

Reserve the baseline model for ambiguous architecture, cross-system diagnosis, synthesis across conflicting evidence, security- or money-sensitive decisions, and final arbitration. A subtask does not deserve the baseline model merely because it writes code or reviews code.

For the currently exposed GPT-5.6 family descriptions, use `luna` first for bounded routine work and `terra` for moderate implementation or analysis when both are confirmed callable and below the baseline. Keep reasoning at the minimum sufficient level and never above the baseline effort. Do not use this example as proof that those models exist in another runtime.

Prefer Luna for a narrowly named function or component, mechanical refactors, explicit-checklist code review, lint/test execution, browser assertions, responsive checks, and defect reproduction. Give Luna the relevant paths and checks, not the complete application context. Escalate to Terra when the work crosses several coupled modules, requires moderate design judgment, or Luna returns a reproducible uncertainty. Escalate to the baseline only for unresolved ambiguity, high-cost risk, or arbitration—not merely because a worker found a bug.

For interaction-heavy or subjective deliverables, budget a direct Luna verification pass when deterministic checks alone cannot prove the result. The reviewer receives the task contract, the produced artifacts, and a narrow evidence rubric; it must operate the result and report reproducible failures. This is a justified lightweight split, not a generic second opinion. Route bounded fixes back to Luna first, then Terra if they require cross-module judgment.

Split by independently verifiable deltas rather than broad roles. For example, `review the whole application` is too broad; `inspect these three changed files for the five listed invariants and write findings to review.md` is suitable for Luna. Likewise, prefer `implement the parser described in task.md and pass test X` over transferring the entire feature history.

For every proposed unit, recommend:

- `model`: the least-capable currently available model that is still sufficient;
- `reasoning_effort`: based on logical complexity, ambiguity, and cost of error, capped by the baseline effort;
- `fast`: based on latency value, extra usage cost, runtime support, and scope.

Prefer stronger capability for architecture, ambiguous diagnosis, safety-critical review, or decisions with expensive failure. Prefer lighter configurations for bounded retrieval, deterministic checks, formatting, and repetitive execution. Do not trade away required correctness merely to reduce tokens.

## Present the confirmation sheet

Give the user a compact proposal containing:

- confirmed available capabilities and any discovery limits;
- whether decomposition is worthwhile;
- a table with task, model, reasoning effort, Fast recommendation and scope, and a short reason;
- execution phases, dependencies, parallel groups, and write-conflict controls;
- a clear stop asking the user to accept, modify, or decline subagents.

Use this shape when helpful:

```text
Confirmed capabilities: ...

| Work unit | Model | Effort | Fast / scope | Why |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

Execution: phase 1 ...; phase 2 ...
Reply with: accept; changes; or do not use subagents.
```

Do not present unavailable models as alternatives. When only one configuration is confirmed, still explain whether splitting is useful and use that configuration consistently.

## Execute only the confirmed plan

After the user accepts or edits the proposal:

- make every execution-agent prompt operationally self-contained but context-minimal: explicitly state that the user confirmed execution, identify the bounded task and accepted configuration, point to source/task artifacts, name allowed writes and checks, and require artifacts to be retained; never rely on inherited pre-confirmation context to convey authorization;
- attach or reference the semantically complete task contract and run the semantic-loss check before dispatch;
- apply exactly the confirmed per-task model and reasoning settings;
- apply Fast only where the interface supports the stated scope;
- warn and obtain separate confirmation before a session-wide Fast change that was not already explicitly accepted;
- preserve task dependencies and isolate conflicting writes;
- restore temporary session-wide state after the work when restoration is supported;
- report any runtime rejection or configuration drift instead of silently substituting another model.
- validate every spawned configuration against the confirmed baseline ceiling immediately before dispatch.
- reject a same-model multi-agent plan unless its confirmation sheet states the concrete risk or elapsed-time benefit that outweighs duplicated context.

Do not mark an interactive or visual task complete from source inspection alone. Exercise each required state transition, capture observable evidence when practical, and compare it with any reference artifact. DOM presence, event-listener presence, syntax success, and “no console errors” are necessary checks but never proof that animation, interaction, data semantics, or visual parity works.

If the user says not to use subagents, continue as a single agent using the user's requested configuration when supported.

## Usage accounting for benchmarks

When measuring a multi-agent Codex run, do not assume `codex exec --json` root usage includes child agents. If local rollout logs are available and the user requests full-tree accounting:

1. Avoid ephemeral execution so child rollouts persist.
2. Reconstruct the tree from spawn events and child `parent_thread_id` values.
3. Read each thread's final cumulative token usage and its model, reasoning effort, and service tier.
4. Sum the entire tree and report root-only stdout usage separately.

For forward-testing this skill, read [references/test-matrix.md](references/test-matrix.md). Keep test artifacts isolated from the user's working files.
