---
id: REQ-2214
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2214

A `Compaction` event MUST NOT supersede a range that an earlier `Compaction`
event already supersedes.

(from docs/spec/session.md [R-SESSION-013], high)
