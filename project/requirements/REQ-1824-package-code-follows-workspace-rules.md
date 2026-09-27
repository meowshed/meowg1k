---
id: REQ-1824
artifact: requirement
topic: pkg
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1824

Code from a package MUST run under the same rules as code the workspace wrote:
the declaration phase reaches nothing, and what a model decides still passes
through policy. A package is Starlark, not a plugin. (from docs/spec/packages.md
[R-PKG-033], high)
