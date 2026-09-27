---
id: REQ-1649
artifact: requirement
topic: llm
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1649

A cancelled request MUST return a distinct cancellation error, never a timeout
or a transport error.

(from docs/spec/llm.md [R-LLM-061], high)
