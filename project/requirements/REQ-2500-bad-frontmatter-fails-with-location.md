---
id: REQ-2500
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2500

Frontmatter that is not valid YAML, or that carries a key outside the set
[R-STAR-050] allows, MUST fail at load time with the file and line. (from
docs/spec/starlark.md [R-STAR-052], high)
