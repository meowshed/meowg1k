---
id: REQ-1648
artifact: requirement
topic: llm
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1648

Every request MUST abort the in-flight HTTP request when its cancellation token
fires.

(from docs/spec/llm.md [R-LLM-060], high)
