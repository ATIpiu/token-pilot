# TokenPilot routing test v3

Evaluator: `gpt-5.6-sol / low`. Each run was plan-only and stopped before execution.

| Environment | Expected routing | Observed routing | Result |
|---|---|---|---|
| Payment double-charge | Sol for money-sensitive diagnosis; Luna for bounded implementation and concurrency verification | Sol/low diagnosis; two Luna/low bounded units | Pass |
| Six localized label edits | One lightweight agent; no Sol duplication | One Luna/low agent for search, edits, and focused tests | Pass |
| Single confirmed configuration | Exactly one Sol/low agent when size and urgency are unknown | Initially failed twice by hypothesizing large documents; after moving the gate to the non-negotiable contract, one Sol/low agent | Pass after correction |
| One variable rename | One agent; no artificial split; remain under ceiling | One Sol/low agent because no lighter model was declared | Pass |

## Skill change justified by the failed case

When only one configuration is confirmed, TokenPilot now requires exactly one agent. Same-model decomposition is allowed only when the user explicitly requests parallel agents, measured inputs exceed one context window, or separate execution environments are technically required. Hypothetical input size and generic latency claims do not qualify.
