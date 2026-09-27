---
id: REQ-2547
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2547

Every hit `search.code` returns MUST carry `path`, `first_line`, `last_line`,
`text` and `score`, so a result can be cited and not only read.

(from crates/meow-star/src/modules.rs:781 and :823, high)
