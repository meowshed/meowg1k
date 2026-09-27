---
id: REQ-1612
artifact: requirement
topic: llm
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1612

A provider MUST deduplicate tool calls that arrive more than once with the same
identifier, before returning the response, regardless of any session or history
setting.

(from docs/spec/llm.md [R-LLM-014], high)
