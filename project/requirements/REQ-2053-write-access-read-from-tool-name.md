---
id: REQ-2053
artifact: requirement
topic: policy
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2053

A tool call MUST reach the policy as a write when the tool's name contains
`write`, `remove`, `append` or `mkdir`, and as a read otherwise.

(from https://github.com/meowshed/meowg1k/pull/127 and
crates/meow-star/src/run.rs:527, high)
