---
id: REQ-2223
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2223

The selectors `@last`, `@last-N`, and `@<agent-name>` MUST resolve at the moment
of use: `@last` to the most recent session in the workspace, `@last-N` to the
Nth most recent, and `@<agent-name>` to the most recent session of that agent.

(from docs/spec/session.md [R-SESSION-032], high)
