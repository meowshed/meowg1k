---
id: REQ-2548
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2548

`search.code` MUST accept `paths`, a list of globs, and return only hits in
files those globs match.

(from crates/meow-star/src/modules.rs:793 and
crates/meow-index/src/index.rs:469, high)
