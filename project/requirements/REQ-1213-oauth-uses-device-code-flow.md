---
id: REQ-1213
artifact: requirement
topic: auth
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1213

A provider kind that authenticates by OAuth rather than an API key MUST obtain
its credential through a device-code flow: the command shows a code and a URL,
waits for the person to approve, and stores the result.

(from docs/spec/auth.md [R-AUTH-020], high)
