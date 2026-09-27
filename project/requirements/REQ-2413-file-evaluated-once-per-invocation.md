---
id: REQ-2413
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2413

A file MUST be evaluated at most once per invocation, whatever the number of
`load` statements naming it. (from docs/spec/starlark.md [R-STAR-007], high)
