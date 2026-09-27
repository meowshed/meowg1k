---
id: REQ-2529
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2529

`toml.encode` MUST fail, saying that TOML has no null, when a value it is given
holds `None`, and never drop the key.

(from https://github.com/meowshed/meowg1k/pull/141 and
crates/meow-star/src/modules.rs:258, high)
