---
id: REQ-1652
artifact: requirement
topic: llm
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1652

The provider transport MUST give up connecting to a vendor after 30 seconds.

(from https://github.com/retran/meowg1k/pull/125 and
crates/meow-llm/src/http.rs:24 and :47, high)
