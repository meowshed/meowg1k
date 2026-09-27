---
id: REQ-1202
artifact: requirement
topic: auth
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1202

Resolution MUST NOT happen while `.meow/` is being evaluated beyond what
`[R-STAR-084]` already allows: a declaration may call `env.get`, and the store
is read when a provider is built rather than when it is declared.

(from docs/spec/auth.md [R-AUTH-003], high)
