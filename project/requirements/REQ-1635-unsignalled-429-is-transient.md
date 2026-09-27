---
id: REQ-1635
artifact: requirement
topic: llm
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1635

When a provider's documented quota signal is absent, a 429 MUST classify as
`Transient`, because retrying a spent quota costs a delay while refusing a rate
limit costs the run.

(from docs/spec/llm.md [R-LLM-037], high)
