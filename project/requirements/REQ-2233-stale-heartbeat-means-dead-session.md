---
id: REQ-2233
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2233

A session whose last event is not `Finished` and whose heartbeat is older than
three intervals MUST be treated as dead.

(from docs/spec/session.md [R-SESSION-043], high)
