---
id: REQ-1639
artifact: requirement
topic: llm
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1639

A provider that does not report caching MUST leave the cached prompt token count
absent, never zero, so that "no cache hits" stays distinguishable from "this
provider does not say".

(from docs/spec/llm.md [R-LLM-040], high)
