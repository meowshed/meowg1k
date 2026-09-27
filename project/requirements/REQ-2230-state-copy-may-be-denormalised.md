---
id: REQ-2230
artifact: requirement
topic: session
class: non-functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2230

A denormalised copy of the session state MAY be kept so that listing a thousand
sessions does not read a thousand events, provided it is rebuildable from the
log and the log wins on any disagreement.

(from docs/spec/session.md [R-SESSION-041], high)
