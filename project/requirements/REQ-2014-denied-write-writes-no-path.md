---
id: REQ-2014
artifact: requirement
topic: policy
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2014

A write that hits a denied path MUST NOT write any path, so the workspace is
never left half-applied.

(from docs/spec/policy.md [R-POLICY-009], high)
