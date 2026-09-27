---
id: REQ-2600
artifact: requirement
topic: store
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2600

The store MUST keep all data for one workspace in a single SQLite database at
`.meow/.data/meow.db`, resolved from the workspace root.

(from docs/spec/store.md [R-STORE-001], high)
