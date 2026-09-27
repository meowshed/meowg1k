---
id: REQ-1203
artifact: requirement
topic: auth
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1203

A runtime module MUST NOT expose the credential store. A handler that wants a
secret reads it from the environment, which is a decision the person running the
command made.

(from docs/spec/auth.md [R-AUTH-004], high)
