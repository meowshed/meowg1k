---
id: REQ-2525
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2525

While `.meow/` is being evaluated, every other runtime module MUST fail with an
error saying it is unavailable during declaration. (from docs/spec/starlark.md
[R-STAR-084], high)
