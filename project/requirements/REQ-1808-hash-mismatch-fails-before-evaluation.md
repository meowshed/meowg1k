---
id: REQ-1808
artifact: requirement
topic: pkg
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1808

Loading MUST fail without evaluating anything when the hash of what it is about
to run and the lockfile differ, naming the package, the expected hash, and the
one found. (from docs/spec/packages.md [R-PKG-011], high)
