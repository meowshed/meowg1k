---
id: REQ-1004
artifact: requirement
topic: agent
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1004

A model response with no tool calls MUST stop the run with `finished`, including
when its text is empty.

(from docs/spec/agent.md [R-AGENT-005], high)
