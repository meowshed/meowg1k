---
id: REQ-1634
artifact: requirement
topic: llm
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1634

A provider MUST distinguish `QuotaExhausted` from a rate limit using its own
documented signal, not by matching text in a message.

(from docs/spec/llm.md [R-LLM-037], high)
