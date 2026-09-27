---
id: BUG-0041
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue: 214
---

# Answering "stop" ends a run `denied` under a `report` policy, and no requirement says so

When the user answers an approval prompt with "stop", the engine ends the run
with stop reason `denied`, whatever the agent's error policy. REQ-1007 names
`denied` for a denial under `abort`, and no requirement describes what "stop"
does.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare an agent with `on_tool_error = "report"` and a policy that asks for
   its one tool.
2. Run it against a recorded reply that calls the tool, with an approver that
   answers `Answer::Stop`.

The run ends `denied`. This was traced through the code and not run.

## What the system does

`crates/meow-agent/src/engine.rs:383` maps `Answer::Stop` to `Some(true)`, and
`crates/meow-agent/src/engine.rs:312-315` returns
`Turn::Stop(StopReason::Denied, ...)` for it before the error policy is read.

## What it should do, and why

No requirement covers it. REQ-2841 makes the prompt offer "stop" and says
nothing of its effect, and REQ-1007 says "`denied` MUST be the stop reason when
the run ends because policy denied a call and the error policy is `abort`".
The behaviour itself is reasonable; the gap is that `spec_first` in `CLAUDE.md`
says no behaviour ships without a requirement. This routes to the requirements
step, which should also settle whether "stop" is `denied` or `cancelled`, because
the user ended the run and the policy didn't refuse anything.

## Triage

The defect enters in the requirements, and the code needs no change until one
is written. It is minor.

## Closed by

A test in `crates/meow-agent/tests/`, named for example
`answering_stop_ends_the_run_whatever_the_error_policy`, citing the new
requirement.
