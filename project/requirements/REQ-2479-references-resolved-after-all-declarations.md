---
id: REQ-2479
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2479

A reference by name MUST be resolved after every declaration file has been
evaluated, so that the order of the declarations it names, inside and between
files, does not matter. A call that takes a value, such as `meow.command`, needs
that value bound first, as any Starlark name does (from
https://github.com/meowshed/meowg1k/pull/123 and
crates/meow-star/src/declare.rs, high). (from
docs/spec/starlark.md [R-STAR-032], high)
