---
id: REQ-2536
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2536

A `meow` declaration builtin MUST treat an explicit `None`, given for an
optional keyword argument whose default is absent, as if the argument were left
out.

(from https://github.com/retran/meowg1k/pull/126 and
crates/meow-star/src/declare.rs:74 and :158, high)
