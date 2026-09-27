---
id: REQ-1074
artifact: requirement
topic: agent
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1074

Requests MUST be counted in the workspace database, so that two invocations
share one ceiling. A limit that resets with the process isn't a limit: run
`meow` in a loop and every invocation starts with a full allowance. A test that
runs two processes and shows the second refused by what the first spent is what
tells a shared limiter from a per-process one.

(from https://github.com/retran/meowg1k/issues/167, high)
