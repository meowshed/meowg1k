---
id: REQ-2553
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2553

`csv.parse` MUST take a `header` argument, `True` by default, saying whether the
first record names the columns.

(from https://github.com/meowshed/meowg1k/pull/141 and
crates/meow-star/src/modules.rs:313, high)
