---
id: REQ-2531
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2531

`meow.on` MUST take `text`, `tool` and `step` as separate callbacks, each
optional, so a handler observes a run without switching on a kind field.

(from docs/design/0.3.0-starlark-api.md section 5.5, high)
