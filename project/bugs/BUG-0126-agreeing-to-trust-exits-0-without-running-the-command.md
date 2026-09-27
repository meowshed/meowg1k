---
id: BUG-0126
artifact: bug
status: approved
severity: major
violates: REQ-2852
found: 2026-09-27
revised: 2026-09-27
issue: 249
---

# Agreeing to trust exits 0 without running the command or saying so

A "y" to the trust question records the agreement, prints "Agreed. `meow trust
--withdraw` undoes it." and exits 0. The command didn't run, and nothing says
so, so `meow review && git commit` commits without a review.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments; `cargo build -p meow-cli`, `HOME` set to
an empty scratch directory.

1. In a workspace with a command `review` that returns `"false"`, run
   `meow review && echo committed` from a terminal.
2. Answer `y` to "Run scripts from this workspace? [y/N]".
3. The shell prints `committed`. `review` never ran.

This follows from the code below; ADR-0460 quotes the same messages.

## What the system does

`ask_to_trust` at `crates/meow-cli/src/wire.rs:221-235` returns `Ending::Passed`
after recording trust, and the message says only how to withdraw it.

## What it should do, and why

REQ-2852 derives exit 0 from "`finished`, and the handler returned true or
nothing". No handler ran, so 0 claims a result that doesn't exist. ADR-0460
says agreeing "says so", which the message doesn't. The binary should either
run the command after the agreement or exit non-zero with a message saying the
command didn't run and to run it again.

## Triage

The defect enters in `ask_to_trust`. It's major: the exit code is the contract
a shell branches on, by the vision's fourth goal, and here it reports success
for work that was never done.

## Closed by

A test in `crates/meow-cli/tests/trust.rs`, named for example
`agreeing_does_not_report_success_for_the_command`, that answers `y` through a
pseudo-terminal and expects a non-zero exit and a message naming the command to
run again, or the command's own output.
