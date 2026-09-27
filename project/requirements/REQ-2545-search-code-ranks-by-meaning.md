---
id: REQ-2545
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2545

`search.code` MUST return at most `limit` hits, 10 by default, ranked against
the question by meaning.

(from docs/design/0.3.0-starlark-api.md section 10 and
crates/meow-star/src/modules.rs:785, high)
