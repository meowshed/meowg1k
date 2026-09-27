---
id: REQ-1604
artifact: requirement
topic: llm
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1604

What a provider sends to authenticate MUST be obtained per request rather than
fixed when the provider is built, so that a credential which expires can be
renewed without rebuilding anything.

(from docs/spec/llm.md [R-LLM-004], high)
