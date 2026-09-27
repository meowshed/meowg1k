---
id: REQ-2524
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2524

While `.meow/` is being evaluated, only `load` and `@std//env` MUST be callable.
(from docs/spec/starlark.md [R-STAR-084], high)

`@std//env` is carved out because credentials are resolved there. (from
docs/spec/starlark.md [R-STAR-084], high)
