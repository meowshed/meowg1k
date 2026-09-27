---
id: REQ-2264
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2264

A `ToolResult` event MAY carry an empty output, recording the call's identifier,
duration and error without what the tool returned.

(from https://github.com/meowshed/meowg1k/pull/129 and
crates/meow-star/src/run.rs:728, high)
