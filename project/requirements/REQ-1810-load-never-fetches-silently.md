---
id: REQ-1810
artifact: requirement
topic: pkg
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1810

Loading MUST NOT fetch silently: a load that reaches the network without being
asked is a load that can change behaviour between two runs of the same commit.
(from docs/spec/packages.md [R-PKG-012], high)
