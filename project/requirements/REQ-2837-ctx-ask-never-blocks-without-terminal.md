---
id: REQ-2837
artifact: requirement
topic: tui
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2837

Any `ctx.ask` call MUST NOT block when stdin is not a terminal or `--yes` was
given.

(from docs/spec/tui.md [R-TUI-051], high)
