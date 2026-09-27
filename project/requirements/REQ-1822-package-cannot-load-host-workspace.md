---
id: REQ-1822
artifact: requirement
topic: pkg
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1822

A file in a package MUST NOT be able to load `//<path>`, which is the host
workspace's own tree. A dependency reaching into the workspace that depends on
it inverts the direction and makes the package's behaviour depend on who loaded
it. (from docs/spec/packages.md [R-PKG-031], high)
