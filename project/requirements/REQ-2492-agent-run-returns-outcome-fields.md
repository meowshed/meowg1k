---
id: REQ-2492
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2492

`agent.run(task, ...)` MUST return a value carrying `text`, `value`, `stop`,
`detail`, `ok`, `usage`, `steps`, and `session`, with `stop` taking one of the
six values in [R-AGENT-002] and `detail` carrying the explanation required by
[R-AGENT-004]. (from docs/spec/starlark.md [R-STAR-042], high)
