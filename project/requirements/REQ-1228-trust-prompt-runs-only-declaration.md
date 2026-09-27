---
id: REQ-1228
artifact: requirement
topic: auth
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1228

Loading `.meow/` to ask the question MUST NOT run any of it beyond declaration,
which `[R-STAR-084]` already guarantees: the prompt describes what was declared,
and declaring reaches nothing.

(from docs/spec/auth.md [R-AUTH-033], high)
