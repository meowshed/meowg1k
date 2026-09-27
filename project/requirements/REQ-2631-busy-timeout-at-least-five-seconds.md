---
id: REQ-2631
artifact: requirement
topic: store
class: non-functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2631

A write that blocks on another writer MUST wait up to a configured busy timeout
of at least 5 seconds before failing.

(from docs/spec/store.md [R-STORE-023], high)
