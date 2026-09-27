---
id: REQ-2538
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2538

`fs.remove` MUST refuse `.meow/.data/` and every path inside it.

(from https://github.com/retran/meowg1k/pull/135 and
crates/meow-star/src/capability.rs:211, high)
