---
id: REQ-2229
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2229

The state MUST be defined by the log: `running` when the last event is not
`Finished`, and otherwise the stop reason that last `Finished` event carries.

(from docs/spec/session.md [R-SESSION-041], high)
