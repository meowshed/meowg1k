---
id: REQ-2467
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2467

A key that was never written MUST give the caller's default, and `None` when it
named none, so that "absent" and "stored `None`" are the same answer only when
the caller asked for that. (from docs/spec/starlark.md [R-STAR-027], high)
