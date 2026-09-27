---
id: REQ-1034
artifact: requirement
topic: agent
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1034

Every run MUST check its cancellation token before each model call, before each
tool call, and while awaiting either.

(from docs/spec/agent.md [R-AGENT-030], high)
