---
id: REQ-2852
artifact: requirement
topic: tui
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2852

The process exit code MUST be derived from the stop reason and the handler's
return value:

| Code | Meaning |
| --- | --- |
| 0 | `finished`, and the handler returned true or nothing |
| 1 | `finished`, and the handler returned false |
| 2 | Usage error: unknown command, bad flag, bad argument |
| 3 | `budget` |
| 4 | `cancelled` |
| 5 | `denied` |
| 6 | Provider or credential failure |
| 9 | `failed` for any other reason, such as a storage error |
| 7 | Configuration error: `.meow/` failed to load |
| 8 | `tool_aborted` |

(from docs/spec/tui.md [R-TUI-080], high)
