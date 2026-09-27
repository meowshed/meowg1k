---
id: REQ-2207
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2207

Once it has ended, each run MUST be closed by exactly one `Finished` event
before the next `Started` event.

(from docs/spec/session.md [R-SESSION-005], high)
