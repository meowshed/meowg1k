---
id: REQ-2630
artifact: requirement
topic: store
class: non-functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2630

A bulk write such as an index build MUST NOT hold the write lock for the length
of the operation, so that indexing cannot block an agent run.

(from docs/spec/store.md [R-STORE-024], high)
