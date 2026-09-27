---
id: REQ-2482
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2482

`meow.model` MUST accept a `kind` of `chat` or `embedding`, defaulting to
`chat`. (from docs/spec/starlark.md [R-STAR-034], high)

The index needs an embedding model and nothing said how a workspace names one,
so [R-INDEX-051] could record which model built an index that no declaration
could choose. (from docs/spec/starlark.md [R-STAR-034], high)
