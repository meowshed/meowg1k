---
id: REQ-2250
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2250

Every tool invocation MUST produce a `Policy` event before the tool runs,
recording the decision, the rule that produced it, and whether the decision came
from a rule, an interactive answer, or a session grant.

(from docs/spec/session.md [R-SESSION-070], high)
