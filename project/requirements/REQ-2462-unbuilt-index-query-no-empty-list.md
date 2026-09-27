---
id: REQ-2462
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2462

A `query` against an index that was never built MUST NOT return an empty list,
because no results and no index are different facts. (from docs/spec/starlark.md
[R-STAR-025], high)
