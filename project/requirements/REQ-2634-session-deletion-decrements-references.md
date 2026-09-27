---
id: REQ-2634
artifact: requirement
topic: store
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2634

The store MUST delete a session by removing its rows and decrementing the
reference count of every blob those rows referenced.

(from docs/spec/store.md [R-STORE-040], high)
