---
id: REQ-1644
artifact: requirement
topic: llm
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1644

A response that fails schema validation MUST be retried with the validation
error included in the follow-up request, up to a configured attempt count.

(from docs/spec/llm.md [R-LLM-051], high)
