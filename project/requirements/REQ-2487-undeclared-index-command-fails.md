---
id: REQ-2487
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2487

An index command in a workspace that declares no index MUST fail saying so
rather than choosing a model. (from docs/spec/starlark.md [R-STAR-035], high)

Choosing a model for somebody is how an index gets built by one model and
queried by another, which [R-INDEX-051] exists to catch after the fact and this
prevents. (from docs/spec/starlark.md [R-STAR-035], high)
