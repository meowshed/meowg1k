---
id: REQ-2239
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2239

Forking at sequence `n` MUST create a new session whose first `n` events are
copies referencing the same blobs.

(from docs/spec/session.md [R-SESSION-052], high)
