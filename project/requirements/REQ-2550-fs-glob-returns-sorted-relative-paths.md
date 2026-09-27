---
id: REQ-2550
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2550

`fs.glob` MUST return, sorted, every file in the workspace whose path relative
to the workspace root matches the pattern.

(from https://github.com/meowshed/meowg1k/pull/135 and
crates/meow-star/src/capability.rs:143, high)
