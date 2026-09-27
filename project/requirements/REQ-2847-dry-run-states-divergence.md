---
id: REQ-2847
artifact: requirement
topic: tui
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2847

The `--dry-run` transcript MUST state that the run diverges from a real one
after the first tool call, because the model's next move depends on a result it
never received.

(from docs/spec/tui.md [R-TUI-072], high)
