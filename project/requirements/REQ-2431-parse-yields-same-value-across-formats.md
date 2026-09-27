---
id: REQ-2431
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2431

`yaml.parse`, `toml.parse`, and `json.parse` MUST produce the same Starlark
value for documents that describe the same data. (from docs/spec/starlark.md
[R-STAR-016], high)
