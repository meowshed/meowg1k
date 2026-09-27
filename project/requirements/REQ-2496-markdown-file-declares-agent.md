---
id: REQ-2496
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2496

A `.md` file under `.meow/agents/` MUST declare an agent whose frontmatter
accepts every keyword argument of `meow.agent` except `system`, plus `include`.
(from docs/spec/starlark.md [R-STAR-050], high)
