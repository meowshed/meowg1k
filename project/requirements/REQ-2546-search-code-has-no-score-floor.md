---
id: REQ-2546
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2546

`search.code` MUST return hits whatever their score, as `index.query` does with
a floor of zero.

(from https://github.com/retran/meowg1k/pull/142 and
crates/meow-cli/src/index.rs:173, high)
