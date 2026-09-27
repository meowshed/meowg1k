---
id: REQ-1046
artifact: requirement
topic: agent
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1046

Compaction MUST NOT drop messages without summarising them, because a silent
drop leaves the model confidently wrong about what it already knows.

(from docs/spec/agent.md [R-AGENT-044], high)
