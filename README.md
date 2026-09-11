# TokenPilot

TokenPilot is a Codex skill for routing bounded implementation, review, and verification work to the least-expensive model configuration that can reliably complete it.

It keeps the current/root model and reasoning effort as a hard ceiling, avoids expensive same-model delegation, and uses semantically complete artifact-based handoffs so lower-tier workers do not lose important interaction or quality requirements.

## What it does

- Discovers the models and reasoning efforts actually exposed by the current runtime.
- Decides whether decomposition is worth its coordination cost.
- Routes routine work to lighter callable models such as Luna or Terra when appropriate.
- Preserves the root model and effort as a hard capability ceiling.
- Requires user confirmation before dispatching a multi-agent execution plan.
- Keeps the root agent responsible for integration and evidence-based acceptance.
- Measures full agent-tree usage in benchmark runs.

## Install

Copy this repository into your Codex skills directory:

```text
~/.codex/skills/token-pilot/
```

Then invoke it explicitly with `$token-pilot`, or ask Codex to optimize model routing for a task.

Example request:

```text
Use TokenPilot. Keep the baseline ceiling at Sol/low. Let Terra handle moderate implementation and Luna handle bounded review and browser verification. Fast off.
```

## Important design rule

The shortest handoff is not necessarily the cheapest handoff. TokenPilot passes the smallest **semantically complete** context: objective, relevant artifact paths, interaction states, invariants, quality bar, and executable acceptance checks.

## Validation

The included test matrix covers single-model environments, model ceilings, direct lightweight execution, coordinator overhead, and large interaction-heavy builds. See [`references/test-matrix.md`](references/test-matrix.md).

The latest local A/B experiment used four interactive web tasks. All four passed operated browser regression after routing three tasks to Terra/low and one to Luna/low. Estimated usage cost fell from `$2.1203` to `$1.4934`—about `29.6%`—while preserving the tested behavior.

Pricing and availability are runtime-dependent. TokenPilot discovers callable configurations instead of assuming a static model catalog.
