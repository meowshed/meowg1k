---
id: REQ-1617
artifact: requirement
topic: llm
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1617

The stream event kinds MUST be exactly: `Text`, `Thinking`, `ToolCallStart`,
`ToolCallDelta`, `ToolCallEnd`, `Usage`, `Error`, and `Done`.

(from docs/spec/llm.md [R-LLM-020], high)
