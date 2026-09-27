---
id: REQ-2639
artifact: requirement
topic: store
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2639

Generation responses MUST NOT be cached unless the caller asks, because an agent
that retries wants a fresh attempt and a cache would hand it the answer that
already failed.

(from docs/spec/store.md [R-STORE-045], high)
