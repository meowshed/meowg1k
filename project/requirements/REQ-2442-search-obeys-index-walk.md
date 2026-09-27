---
id: REQ-2442
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2442

`search.text` and `search.files` MUST obey the same walk the index obeys, so a
file the index ignores is a file they do not report. (from docs/spec/starlark.md
[R-STAR-019], high)
