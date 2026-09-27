---
id: REQ-2007
artifact: requirement
topic: policy
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2007

A tool MUST NOT resolve the path a second time. Re-resolving reopens the window
in which a path allowed as a file becomes a symlink to somewhere denied.

(from docs/spec/policy.md [R-POLICY-014], high)
