---
id: REQ-2541
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2541

`@std//git` MUST refuse a revision or a path a caller supplies that begins with
`-`.

(from https://github.com/retran/meowg1k/pull/136 and
crates/meow-star/src/capability_git.rs:50, high)
