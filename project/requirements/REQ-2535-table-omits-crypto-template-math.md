---
id: REQ-2535
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: static
---

# REQ-2535

The module table MUST NOT contain `crypto`, `template` or `math`.

(from docs/design/0.3.0-starlark-api.md section 10 and
crates/meow-star/src/modules.rs:35, high)
