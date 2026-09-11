# TokenPilot Forward-Test Report

Date: 2026-09-11

Evaluator baseline for all runs:

- Model: `gpt-5.6-sol`
- Reasoning effort: `low`
- Sandbox: read-only
- Execution: ephemeral; no child agents were spawned

## Results

| Environment | Key behavior observed | Result |
|---|---|---|
| Full per-agent controls | Split the high-risk payment task into diagnosis, design, implementation, concurrency tests, and independent review; used stronger reasoning for ambiguous/high-risk work; gave one agent exclusive write ownership; labeled Fast per task. | Pass |
| Session-wide Fast only | Chose a single agent for the short overlapping edit flow; labeled Fast as whole-session scope; stopped for acceptance or declining Fast without switching it. | Pass |
| Restricted single model | Explicitly stated incomplete discovery; used only `gpt-5.6-sol / low`; marked Fast unavailable; chose one agent for coherent synthesis. | Pass |
| Task not worth splitting | Chose one `gpt-5.6-sol / low` agent; avoided artificial delegation; proposed per-task Fast and stopped before edits/tests. | Pass |

## Shared invariant audit

- Declared capabilities only: pass in 4/4.
- Explicit decomposition decision: pass in 4/4.
- Model, effort, Fast, and scope recommendation: pass in 4/4.
- Dependencies/write-conflict handling where relevant: pass in 4/4.
- Stopped before execution or configuration mutation: pass in 4/4.
- No Fast/model/reasoning conflation: pass in 4/4.

No corrective skill change was justified by these runs. Raw final responses are stored in the workspace `output/token-pilot-env-1.txt` through `output/token-pilot-env-4.txt`.
