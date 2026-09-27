---
id: REQ-2466
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2466

`store.get` MUST return the value that `store.put` was given, of the same type,
for any value a handler can build. (from docs/spec/starlark.md [R-STAR-027],
high)
