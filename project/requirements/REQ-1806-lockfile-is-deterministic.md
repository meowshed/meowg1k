---
id: REQ-1806
artifact: requirement
topic: pkg
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1806

The lockfile MUST be deterministic: the same declarations and the same upstream
produce the same bytes, so a diff shows a dependency change and nothing else.
(from docs/spec/packages.md [R-PKG-010], high)
