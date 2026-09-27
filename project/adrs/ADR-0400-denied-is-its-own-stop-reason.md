---
id: ADR-0400
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1001, REQ-1007, REQ-1008]
supersedes: []
---

# 0400. A run a policy stopped ends with `denied`, apart from `tool_aborted`

## Decision

`denied` stays a stop reason of its own, distinct from `tool_aborted` (from
https://github.com/meowshed/meowg1k/pull/109, high). The engine stops with
`denied` rather than `tool_aborted` when the boundary held rather than something
breaking (from https://github.com/meowshed/meowg1k/pull/119, high).

Once this is accepted, `StopReason` carries six variants, the engine returns
`Denied` for a refusal and `ToolAborted` for a failing tool under an `abort`
policy, and the command line maps them to exit codes 5 and 8 (from
crates/meow-core/src/event.rs:18-31, crates/meow-agent/src/engine.rs:312-326,
crates/meow-cli/src/exit.rs:45-48, high). Nothing in this decision is left
unbuilt (reasoned from the same files, low).

## Why

A user needs to know the boundary held rather than that something broke, and
exit code 5 already assumed that distinction (from
https://github.com/meowshed/meowg1k/pull/109, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: keep the first design's four stop reasons, `finished`, `budget`, `cancelled` and `tool_aborted`, so a refusal ends as `tool_aborted` (from docs/design/0.3.0-architecture.md and docs/design/0.3.0-starlark-api.md at commit 90d0007, high) | One fewer variant for every consumer of the stop reason to handle, and one fewer exit code (reasoned from crates/meow-core/src/event.rs:11-15, low) | A user can't tell that the boundary held from something breaking, and exit code 5 already assumed the distinction (from https://github.com/meowshed/meowg1k/pull/109, high) |

The history names only this one alternative, which is also the prior state, so
no third option exists to compare (from
https://github.com/meowshed/meowg1k/pull/109, high).

## What it costs

Adding the sixth variant is a change to three specifications at once, because
the session's terminal state and the exit-code table reuse the stop reason set
(from crates/meow-core/src/event.rs:11-15, high). Every consumer, the exit-code
table, the session state and `r.stop` in Starlark, handles one more case
(reasoned from crates/meow-cli/src/exit.rs:38-54, low).

## What would reverse it

- The exit-code table stops giving `denied` and `tool_aborted` separate codes,
  so no script can branch on the difference (reasoned from
  crates/meow-cli/src/exit.rs:45-48, low).

## Consequences

- Exit code 5 means a policy stop and exit code 8 means a tool failure, and the
  match in `code` is total so no two reasons share a number (from
  crates/meow-cli/src/exit.rs:32-54, high).
- An approval answered "stop" also ends the run with `denied`, whatever the
  tool-error policy says, because the person is ending the run and not
  reporting a tool failure (from crates/meow-agent/src/engine.rs:312-315 and
  crates/meow-agent/src/engine.rs:452-455, high).

## How I will know it was realised

1. `a_denial_stops_with_denied_and_a_failure_with_tool_aborted` in
   crates/meow-agent/tests/spec.rs passes: a refused call ends with `Denied`
   and a failing tool under `abort` ends with `ToolAborted` (from
   crates/meow-agent/tests/spec.rs:803-830, high).
2. `the_codes_are_the_ones_the_table_names` in crates/meow-cli/src/exit.rs
   passes, asserting codes 5 and 8 (from crates/meow-cli/src/exit.rs:115-127,
   high).

## What this does not settle

- Which stop reason a policy refusal under a `report` policy produces: the run
  continues, so none does (from crates/meow-agent/src/engine.rs:303-306, high).
- That an approval answered "stop" yields `denied` under a `report` policy is
  what the code does, and no requirement states it: REQ-1007 covers only the
  `abort` policy (from crates/meow-agent/src/engine.rs:312-315 and
  docs/requirements/REQ-1007-policy-denial-stops-denied.md, medium).
