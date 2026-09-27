---
id: REQ-1206
artifact: requirement
topic: auth
class: non-functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1206

`meow auth` MUST refuse to read a credential store file that is readable by
anyone else, naming the file and the permission it expects.

(from docs/spec/auth.md [R-AUTH-011], high)
