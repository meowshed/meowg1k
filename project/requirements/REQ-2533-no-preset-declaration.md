---
id: REQ-2533
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2533

The `meow` global MUST NOT offer a preset declaration, because a named
`meow.model` carrying its own temperature is what a v0.2.x preset named.

(from docs/design/0.3.0-starlark-api.md sections 4 and 11 and
crates/meow-star/src/declare.rs:183, high)
