---
id: REQ-1823
artifact: requirement
topic: pkg
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1823

A package MUST NOT be able to load another package the host workspace did not
declare. Dependencies are the workspace's to state, so that `meow.lock` is the
whole list. (from docs/spec/packages.md [R-PKG-032], high)
