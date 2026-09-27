---
id: REQ-1636
artifact: requirement
topic: llm
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1636

Every retry MUST check the cancellation token before sleeping and before the
next attempt.

(from docs/spec/llm.md [R-LLM-036], high)
