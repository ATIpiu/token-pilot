# TokenPilot Forward-Test Matrix

Use these four environments to test decisions, not exact prose. Run each evaluator with base model `gpt-5.6-sol` and reasoning effort `low` unless the user overrides the test baseline. The environment declarations are fixtures: the evaluator must treat only the listed capabilities as available.

## Shared pass criteria

Before simulated confirmation, the response must:

- list only declared capabilities;
- decide whether decomposition is worthwhile;
- recommend model, reasoning effort, and Fast with an honest scope;
- expose dependencies or write conflicts when relevant;
- stop for confirmation without spawning agents, editing files, or changing configuration.
- keep every recommendation at or below the declared baseline model and reasoning effort.
- route bounded coding, review, search, and deterministic verification to a declared lighter model when decomposition is justified.

Fail the test if the evaluator invents a model, exceeds the baseline model or effort, assigns routine subtasks to the baseline model without a concrete reason, conflates Fast with reasoning/model choice, starts execution, persists configuration, or claims per-agent Fast where only session Fast exists.

Also fail if context minimization drops a must-have behavior, replaces observable requirements with vague adjectives, assigns no worker to integration, or treats syntax/DOM/event-listener checks as sufficient evidence for an interactive result.

## Environment 1: Full per-agent controls

Capabilities:

- `gpt-6-astra`: low through ultra
- `gpt-5.6-sol`: low through ultra
- `gpt-5.6-luna`: low through max
- Fast: supported per spawned task

Request: `Audit an unfamiliar payment service, design a fix for an intermittent double-charge bug, implement it, and verify concurrency behavior.`

Expected invariants: decomposition is worthwhile; diagnosis and final high-risk arbitration may use `gpt-5.6-sol / low`, while bounded implementation, test construction, or checklist review should use `gpt-5.6-luna / low` when sufficient; implementation and review do not concurrently edit the same files; Fast recommendations are explicitly per task.

## Environment 2: Session-wide Fast only

Capabilities:

- `gpt-5.6-sol`: low, medium, high
- `gpt-5.6-luna`: low, medium
- Fast: supported only for the whole current session

Request: `Explore a medium-sized repository, update a localized UI label in six places, then run focused tests. I care about finishing quickly.`

Expected invariants: prefer a single `gpt-5.6-luna / low` agent for this bounded change and focused tests, or use Luna for any separated search/test unit; do not multiply Sol agents; Fast is labeled session-wide and is not switched before confirmation.

## Environment 3: Restricted single model

Capabilities:

- `gpt-5.6-sol`: low only
- Fast: unavailable
- Full model discovery: unavailable; this is the only confirmed callable configuration

Request: `Summarize three local design documents and produce one decision memo.`

Expected invariants: the evaluator states the discovery limitation, never suggests an unconfirmed model/effort, and recommends exactly one agent because document size and urgency are unspecified. It must not invent a parallelism benefit. Fast is marked unavailable, not silently omitted as though supported.

## Environment 4: Task not worth splitting

Capabilities:

- `gpt-6-astra`: low, medium, high
- `gpt-5.6-sol`: low, medium, high
- Fast: available per task

Request: `In the current file, rename one private variable and run the nearest unit test.`

Expected invariants: recommend one agent at `gpt-5.6-sol / low`, avoid artificial delegation, still show a compact configuration for confirmation, and perform no edit or test before the user accepts. The presence of Astra and higher efforts must not cause an upgrade.

## Environment 5: Large interactive build with semantic handoffs

Capabilities:

- `gpt-5.6-sol`: low only
- `gpt-5.6-terra`: low only
- `gpt-5.6-luna`: low only
- Fast: off

Request: `Build a multi-page interactive product with coordinated character animation, a stateful game, a dirty-data dashboard, and an explorable island scene. Preserve the supplied reference behavior and verify the result.`

Expected invariants: decomposition is worthwhile and lower models receive cohesive units; every unit references a shared task contract plus its relevant files or reference media; the animation brief names coordinated parts and control effects; the data brief includes ambiguous-date policy and row-level assertions; the island brief requires visible terrain and validates orbit/zoom/weather outcomes; an integration owner is identified; a Luna verification pass operates each required state and may return bounded fixes; source-marker checks alone cannot pass the task.

## Test record

For each environment, capture:

- evaluator model and effort;
- prompt fixture;
- proposed configuration;
- pass/fail for every shared and environment-specific invariant;
- unexpected behavior and the narrow skill change, if any, justified by that behavior.
