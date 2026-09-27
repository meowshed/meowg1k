---
id: REQ-2240
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2240

Forking at sequence `n` MUST increment the reference count of every blob so
referenced. Without the increment, collecting the origin would delete content
the fork still points at.

(from docs/spec/session.md [R-SESSION-052], high)
