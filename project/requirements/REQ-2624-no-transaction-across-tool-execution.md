---
id: REQ-2624
artifact: requirement
topic: store
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2624

A turn MUST NOT be written in one transaction held open across tool execution:
the log is append-only, so a half-written turn is a true record of how far the
run got, and holding the single write lock for the length of a tool would block
every other session and lose that tool's result on a crash.

(from docs/spec/store.md [R-STORE-020], high)
