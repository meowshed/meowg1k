---
id: REQ-1651
artifact: requirement
topic: llm
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1651

A streaming request that a vendor answers with a non-success status MUST fail at
the call, never through the stream the call would have returned.

(from https://github.com/meowshed/meowg1k/pull/125 and
crates/meow-llm/src/http.rs:129, high)
