---
id: REQ-2231
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2231

While a run is in flight its writer MUST update a heartbeat timestamp on the
session row at a fixed interval.

(from docs/spec/session.md [R-SESSION-043], high)
