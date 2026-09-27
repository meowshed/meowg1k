---
id: REQ-1208
artifact: requirement
topic: auth
class: non-functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1208

A write of the store MUST NOT leave a truncated file if the process dies,
because a truncated credential store locks the user out of every provider at
once.

(from docs/spec/auth.md [R-AUTH-012], high)
