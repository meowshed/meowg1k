---
id: REQ-1062
artifact: requirement
topic: agent
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1062

When a sink declines delta events, the engine MUST deliver the completed text
once per step instead, because a sink backed by a script callback would
otherwise pay one call per token.

(from docs/spec/agent.md [R-AGENT-073], high)
