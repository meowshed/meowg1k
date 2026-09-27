---
id: REQ-2474
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2474

`path` MUST NOT return `\`, so that a path `path.join` built and a path
`search.files` reported can be compared, and a `.meow/` written once behaves the
same everywhere. (from docs/spec/starlark.md [R-STAR-029], high)
