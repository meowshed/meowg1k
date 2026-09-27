---
id: REQ-1816
artifact: requirement
topic: pkg
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1816

The cache MUST be keyed by hash, so two workspaces pinning the same package
share one copy and a changed pin cannot be served the old bytes. (from
docs/spec/packages.md [R-PKG-021], high)
