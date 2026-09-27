---
id: REQ-2495
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2495

`meow.parallel` MUST reject anything that is not an invocation, including a
Starlark function, with an error explaining that Starlark values cannot cross a
thread boundary. (from docs/spec/starlark.md [R-STAR-044], high)
