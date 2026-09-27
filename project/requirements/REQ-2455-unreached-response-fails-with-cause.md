---
id: REQ-2455
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2455

A call that never reached a response - a name that does not resolve, a refused
connection, a deadline - MUST fail with what went wrong. (from
docs/spec/starlark.md [R-STAR-023], high)
