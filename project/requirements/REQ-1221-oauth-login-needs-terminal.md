---
id: REQ-1221
artifact: requirement
topic: auth
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1221

`meow auth login` for an OAuth provider MUST be refused when there is no
terminal, because there is nobody to read the code.

(from docs/spec/auth.md [R-AUTH-023], high)
