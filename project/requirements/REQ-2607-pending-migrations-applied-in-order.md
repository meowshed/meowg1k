---
id: REQ-2607
artifact: requirement
topic: store
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2607

On open, the store MUST apply every migration whose version is greater than the
recorded one, in ascending order, each inside its own transaction.

(from docs/spec/store.md [R-STORE-004], high)
