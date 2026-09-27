---
id: ADR-2805
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2852, REQ-2853, REQ-2854]
supersedes: []
---

# 2805. Each stop reason has its own exit code

## Decision

The exit code comes from the stop reason and the handler's return value, and no
code is reused for a different stop reason (from docs/spec/tui.md [R-TUI-080]
[R-TUI-081], high). The source's account of the v0.2.x gap doesn't cite these
requirements; I mapped them, and the code confirms the mapping: `exit::code`
follows the table of [REQ-2852] arm by arm and repeats no number (from
crates/meow-cli/src/exit.rs:32-55, high).

Once this holds, `meow review --strict && git push` works as a gate, and a CI
job can tell "the budget ran out" from "the code is clean" (from
docs/design/0.3.0-tui.md section 2.4, high). A command run from a workspace
reaches only 0, 1 and 9 today: the binary turns a handler's return into 0 or 1
and any handler error into 9, and never consults the stop reason of an
`agent.run` inside the handler (from crates/meow-cli/src/wire.rs:1162-1180,
medium).

## Why

v0.2.x has no exit code beyond success and failure, so an agent can't act as a
gate (from docs/spec/tui.md, high). A shell branches on the code, which is why
no code is reused (from docs/spec/tui.md [R-TUI-081], high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: success and failure only, as in v0.2.x | Every shell and CI system already reads zero as success and anything else as failure, so there is no table to learn (reasoned from docs/design/0.3.0-tui.md section 2.4, low) | An agent can't act as a gate (from docs/spec/tui.md, high), and a CI job reads "the budget ran out" as "the code is clean" (from docs/design/0.3.0-tui.md section 2.4, high) |
| One code shared by `denied` and `tool_aborted` | Fewer codes for a script to handle (reasoned from https://github.com/retran/meowg1k/pull/109, low) | A user needs to know the boundary held, not that something broke, and code 5 already assumed that distinction (from https://github.com/retran/meowg1k/pull/109, high) |
| The table without a code for `failed`, as drafted before commit e39cd2f | A shorter table (reasoned from docs/design/0.3.0-tui.md section 2.4, low) | A stop reason with no code breaks [REQ-2854], so commit e39cd2f added 9 for `failed` (from git show e39cd2f -- docs/spec/tui.md, high) |

## What it costs

The table is a public contract: a script that branches on 3 for `budget` breaks
if the number moves (from docs/spec/tui.md [R-TUI-081], high). Adding a stop
reason means adding a code, and the unit test's `REASONS` list has to grow with
it (from crates/meow-cli/src/exit.rs:82-91, high). `finished` shares 0 with a
handler that passed, by design, so 0 alone doesn't say whether a handler was
asked (from crates/meow-cli/src/exit.rs:50-53, high).

## What would reverse it

A stop reason added to [R-AGENT-002] that no free code can take without
colliding with the codes a shell reserves for itself, 126 and above, would force
a different scheme (reasoned from crates/meow-cli/src/exit.rs, low).

## Consequences

Usage errors exit 2, a workspace that won't load exits 7, and a missing
credential exits 6 (from crates/meow-cli/tests/surface.rs, high). A handler
that returns false exits 1, and one that returns nothing or true exits 0 (from
crates/meow-cli/src/exit.rs:57-67, high).

## How I will know it was realised

1. `every_stop_reason_has_a_code_of_its_own`,
   `the_codes_are_the_ones_the_table_names` and
   `a_handler_that_returns_nothing_has_not_failed` in
   crates/meow-cli/src/exit.rs pass.
2. `the_handlers_verdict_decides_between_zero_and_one`,
   `a_usage_mistake_exits_two`, `a_broken_workspace_exits_seven` and
   `a_missing_credential_exits_six` in crates/meow-cli/tests/surface.rs pass
   against the built binary.
3. Not yet realised: no test runs the binary to a `budget`, `cancelled`,
   `denied` or `tool_aborted` stop and reads 3, 4, 5 or 8, and `exit::of`,
   which maps an `Outcome` to an ending, has no caller (from
   crates/meow-cli/src/exit.rs:70-75 and crates/meow-cli/src/wire.rs:1162-1180,
   medium).

## What this does not settle

- Which stop reason a handler's run should report when the handler calls
  `agent.run` several times, or ignores the outcome it gets back as a dict with
  `stop` and `ok` (from crates/meow-star/src/value.rs:183-191, medium).
- Whether a provider failure in the middle of a run exits 6 or 9: the binary
  returns 6 only for failures before the handler starts (from
  crates/meow-cli/src/wire.rs:1066-1080, medium).
