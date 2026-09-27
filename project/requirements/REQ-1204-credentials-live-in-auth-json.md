---
id: REQ-1204
artifact: requirement
topic: auth
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1204

Credentials MUST live in one file at `~/.meow/auth.json`, outside every
workspace, so that a repository cannot carry one and a contributor cannot commit
one.

(from docs/spec/auth.md [R-AUTH-010], high)
