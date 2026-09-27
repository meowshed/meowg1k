---
id: REQ-2234
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2234

Opening a session whose last event is not `Finished` and whose recording process
is no longer alive MUST append `Finished { stop: failed, reason: "process
exited" }`.

(from docs/spec/session.md [R-SESSION-042], high)
