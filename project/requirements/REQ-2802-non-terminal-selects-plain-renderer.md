---
id: REQ-2802
artifact: requirement
topic: tui
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2802

A stdout that is not a terminal, or `NO_COLOR` in the environment, or
`--color=never`, MUST select the plain renderer.

(from docs/spec/tui.md [R-TUI-003], high)
