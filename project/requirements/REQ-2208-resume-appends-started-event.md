---
id: REQ-2208
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2208

Resuming a session MUST append a new `Started` event, so that a session that has
already ended can be continued without rewriting the event that ended it.

(from docs/spec/session.md [R-SESSION-006], high)
