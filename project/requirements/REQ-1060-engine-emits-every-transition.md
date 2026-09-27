---
id: REQ-1060
artifact: requirement
topic: agent
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1060

The engine MUST emit every observable transition to its `Sink`: run start
and end, step start and end, text and thinking deltas, tool call start and end,
policy decisions, and usage.

(from docs/spec/agent.md [R-AGENT-070], high)
