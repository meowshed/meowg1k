---
id: REQ-2534
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2534

`env.require` MUST fail while the workspace loads, naming the variable, when
that variable is unset.

(from docs/design/0.3.0-starlark-api.md section 4 and
crates/meow-star/src/modules.rs:123, high)
