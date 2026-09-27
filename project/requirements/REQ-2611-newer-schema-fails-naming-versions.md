---
id: REQ-2611
artifact: requirement
topic: store
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2611

On opening a database whose recorded schema version is greater than the binary
supports, the store MUST fail with an error naming both versions.

(from docs/spec/store.md [R-STORE-006], high)
