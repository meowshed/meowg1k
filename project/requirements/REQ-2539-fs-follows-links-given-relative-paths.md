---
id: REQ-2539
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2539

`fs` MAY follow a symbolic link inside the workspace that points outside it,
when the call gives a relative path.

(from https://github.com/meowshed/meowg1k/pull/135 and
crates/meow-star/src/capability.rs:44, high)
