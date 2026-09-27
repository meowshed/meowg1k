---
id: BUG-0106
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue: 229
---

# A dry run states its divergence once per planned tool, not once per run

`--dry-run` warns that the run has diverged the first time each planned tool is
called, so a run that calls two tools gets the warning twice, and the second
one arrives after the divergence already happened.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare an agent with two tools, `a` and `b`.
2. Run it with `--dry-run` against a scripted model that calls `a`, then `b`.
3. The transcript holds the warning "this is a dry run: from here on ..." twice.

This follows from the code below; ADR-0431 names the missing test.

## What the system does

Each `Planned` tool carries its own `warned` flag,
`crates/meow-star/src/run.rs:387-394` and `:797`, and `Planned::call` warns when
its own flag is unset, `crates/meow-star/src/run.rs:830-837`. Every tool set a
sub-agent builds gets fresh flags as well.

## What it should do, and why

No requirement states how often. REQ-2847 asks the transcript to "state that the
run diverges from a real one after the first tool call"; ADR-0431 says it's
stated once. The warning should appear once, at the first planned call in the
run.

## Triage

The defect enters where `tools_for` builds `Planned` tools. No requirement
covers the count, which is a gap for the requirements step to close by making
REQ-2847 say once per run. It's minor: the warning is shown, only repeated.

## Closed by

A test in `crates/meow-star/tests/`, named for example
`a_dry_run_with_two_tools_warns_once`, that runs the scenario above and counts
one warning.
