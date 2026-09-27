---
id: REQ-1000
artifact: requirement
topic: agent
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1000

Every run that starts MUST return an `Outcome`. No stop condition,
cancellation included, may be reported as an error, because an error discards
the transcript the run already paid for.

(from docs/spec/agent.md [R-AGENT-001], high)
