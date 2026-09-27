---
id: REQ-1653
artifact: requirement
topic: llm
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1653

The provider transport MUST NOT put a deadline on a whole response, because a
model thinking for two minutes is normal.

(from https://github.com/retran/meowg1k/pull/125 and
crates/meow-llm/src/http.rs:46, high)
