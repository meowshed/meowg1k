---
id: ADR-0460
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1223]
supersedes: []
---

# 0460. Agreeing to trust a workspace doesn't then run the command

## Decision

Agreeing doesn't run the command: it says so and exits zero (from
https://github.com/retran/meowg1k/pull/154, high).

Once this is accepted, a "y" to "Run scripts from this workspace? [y/N]" records
the agreement, prints "Agreed. `meow trust --withdraw` undoes it." and exits 0
without running the command; a "no" prints "Not agreed, so nothing ran." and
exits 7 (from crates/meow-cli/src/wire.rs:221, high). The message after a yes
doesn't say that the command didn't run or that it has to be run again, which
the decision's "it says so" promises (from crates/meow-cli/src/wire.rs:228,
high).

## Why

The person answered a question about trust, not about whether to run this, and
starting an agent off the back of a yes is the more surprising of the two (from
https://github.com/retran/meowg1k/pull/154, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Run the command once the person agrees | The person gets what they typed with one answer and no second invocation (reasoned from crates/meow-cli/src/wire.rs:226, low) | It is the more surprising outcome, because the question was about trust (from https://github.com/retran/meowg1k/pull/154, high) |
| Do nothing: never ask at the terminal, and always refuse with "Run `meow trust` here" as a run with no terminal does | One path for every run, and agreeing is always a deliberate command (reasoned from crates/meow-cli/src/wire.rs:213, low) | REQ-1223 requires the first invocation to ask once (from docs/spec/auth.md [R-AUTH-030], high) |

The code and the history name no third option (from
https://github.com/retran/meowg1k/pull/154, medium).

## What it costs

The person runs the command a second time after agreeing (from
crates/meow-cli/src/wire.rs:229, high). The first invocation exits 0 although
the command didn't run, so a shell line such as `meow review && git push` goes
on to its next step (reasoned from crates/meow-cli/src/exit.rs:40, low).

## What would reverse it

People are observed answering yes and then taking the exit 0 as the command's
result, or the prompt gains a separate question about running now (reasoned
from crates/meow-cli/src/wire.rs:232, low).

## Consequences

- Agreeing at the prompt and running `meow trust` leave the same record, and
  neither runs anything (from crates/meow-cli/src/wire.rs:226 and
  crates/meow-cli/src/wire.rs:313, high).
- A record that fails to write exits 7 with the error, so an agreement is never
  assumed without being stored (from crates/meow-cli/src/wire.rs:234, high).

## How I will know it was realised

No test answers the prompt, because every test in
`crates/meow-cli/tests/trust.rs` runs with standard input closed and so takes
the no-terminal path (from crates/meow-cli/tests/trust.rs:30, high). A test
would need a pseudo-terminal that answers "y", then assert exit 0, the "Agreed"
line, no handler output and a trust record (reasoned from
crates/meow-cli/src/wire.rs:221, low).

## What this does not settle

- Whether the exit code after agreeing should differ from a completed run's
  (reasoned from crates/meow-cli/src/exit.rs:40, low).
- The wording after a yes, which doesn't yet tell the person to run the command
  again (from crates/meow-cli/src/wire.rs:228, high).
