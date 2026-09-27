---
id: REQ-2616
artifact: requirement
topic: store
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2616

The same hash MUST return the same bytes whether its payload is kept inline or
in the `blobs` table.

(from docs/spec/store.md [R-STORE-010], high)
