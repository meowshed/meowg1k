---
id: REQ-1200
artifact: requirement
topic: auth
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1200

A credential MUST be resolved in this order: the `api_key` the declaration
gives, then the global store, then the provider's environment variable. The
first that is present and not empty wins.

(from docs/spec/auth.md [R-AUTH-001], high)
