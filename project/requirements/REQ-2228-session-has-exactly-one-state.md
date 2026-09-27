---
id: REQ-2228
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2228

A session MUST be in exactly one state: `running`, or one of the terminal states
`finished`, `budget`, `cancelled`, `denied`, `tool_aborted`, or `failed`.

(from docs/spec/session.md [R-SESSION-040], high)
