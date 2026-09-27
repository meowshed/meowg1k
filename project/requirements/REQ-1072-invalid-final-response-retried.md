---
id: REQ-1072
artifact: requirement
topic: agent
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1072

A final response that fails schema validation MUST be retried per [R-LLM-051]
before the run stops with `failed`.

(from docs/spec/agent.md [R-AGENT-081], high)
