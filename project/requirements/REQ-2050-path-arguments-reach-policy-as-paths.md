---
id: REQ-2050
artifact: requirement
topic: policy
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2050

A tool call MUST reach the policy with the values of its arguments named `path`,
`file`, `paths` and `files` as the paths it touches, each resolved against the
workspace root.

(from https://github.com/meowshed/meowg1k/pull/127 and
crates/meow-star/src/run.rs:540, high)
