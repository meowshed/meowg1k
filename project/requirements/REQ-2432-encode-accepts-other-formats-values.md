---
id: REQ-2432
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2432

Each `encode` of `yaml`, `toml`, and `json` MUST accept any value the other two
produce that its own format can represent. TOML has no null, so `toml.encode`
is the one encoder that meets a value it can't represent (from
https://github.com/meowshed/meowg1k/pull/141, high). (from docs/spec/starlark.md [R-STAR-016], high)
