---
id: BUG-0027
artifact: bug
status: approved
severity: major
violates: REQ-2852
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A workspace command exits only 0, 1 or 9, whatever the agent's stop reason

A workspace command's exit code comes from the handler's return value alone. An
`agent.run` inside the handler that stops on `budget`, `denied`, `cancelled` or
`tool_aborted` doesn't reach the exit code, and `exit::of`, which maps a stop
reason to a code, has no caller.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Write a command whose handler runs an agent with `budget = meow.budget(steps
   = 1)` against a model that asks for a tool, and returns the outcome's text.
2. Run the command and read `$?`.

It exits 0. The budget stop should exit 3. This was traced through the code and
not run.

## What the system does

`crates/meow-cli/src/wire.rs:1162-1179` maps `Ok(Ok(returned))` to
`exit::finished(verdict(&returned))`, which yields 0 or 1, and every error to
`Ending::Stopped(StopReason::Failed)`, which is 9. `exit::of` at
`crates/meow-cli/src/exit.rs:69-75` would map the stop reason, and
`grep -rn "exit::of" crates` finds no caller.

## What it should do, and why

REQ-2852: "The process exit code MUST be derived from the stop reason and the
handler's return value", with 3 for `budget`, 4 for `cancelled`, 5 for
`denied` and 8 for `tool_aborted`. REQ-2854: "Every stop reason in
[R-AGENT-002] MUST map to exactly one exit code." The vision's fourth goal wants
an exit code a shell can branch on.

## Triage

The defect enters in `meow-cli`'s command wiring, which never learns the stop
reason of the runs a handler made. It is major because four of the table's
codes are never produced. The requirement doesn't say which stop reason decides
when a handler runs several agents, and that choice routes to the requirements
step before the fix.

## Closed by

A test in `crates/meow-cli/tests/`, named for example
`a_budget_stop_inside_a_command_exits_3`, that runs the command above against a
recorded provider and expects exit status 3.
