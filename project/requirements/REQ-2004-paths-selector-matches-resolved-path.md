---
id: REQ-2004
artifact: requirement
topic: policy
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2004

A `paths` selector MUST match against an absolute path with symlinks already
resolved, using glob semantics where `**` crosses directory boundaries.

(from docs/spec/policy.md [R-POLICY-003], high)
