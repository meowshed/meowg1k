---
id: REQ-2032
artifact: requirement
topic: policy
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2032

For every call, the effective decision MUST be the more restrictive of what the
workspace policy and the agent policy say, ordering `deny` above `ask` above
`allow`.

(from docs/spec/policy.md [R-POLICY-030], high)
