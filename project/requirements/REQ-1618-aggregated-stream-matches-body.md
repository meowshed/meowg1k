---
id: REQ-1618
artifact: requirement
topic: llm
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1618

Aggregating a recorded stream MUST produce the same response value as parsing
the non-streaming body for the same model output.

Stating it over a recording rather than over two live calls is what makes it
checkable.

(from docs/spec/llm.md [R-LLM-021], high)
