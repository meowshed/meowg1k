---
id: REQ-2445
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2445

The handler context MUST expose exactly six members: `args`, `session`, `out`,
`ask`, `stdin`, and `workspace`, plus `cancelled()`. (from docs/spec/starlark.md
[R-STAR-020], high)
