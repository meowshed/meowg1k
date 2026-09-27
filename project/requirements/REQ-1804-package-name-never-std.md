---
id: REQ-1804
artifact: requirement
topic: pkg
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1804

A package name MUST NOT be `std`. The scheme that reaches the runtime modules
cannot be shadowed by something fetched. (from docs/spec/packages.md
[R-PKG-003], high)
