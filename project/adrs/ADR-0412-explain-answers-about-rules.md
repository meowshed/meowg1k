---
id: ADR-0412
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2046, REQ-2047]
supersedes: []
---

# 0412. `meow policy explain` answers about rules, not outcomes

## Decision

`explain` answers about rules, not outcomes, and doesn't claim to predict how an
`ask` would be answered (from https://github.com/retran/meowg1k/pull/119, high).
`meow_policy::explain` evaluates the call with an empty set of grants and
returns the rule decision, the rule, where it was written and how many
higher-precedence rules were checked (from
crates/meow-policy/src/prompt.rs:135-167, high).

Once this is accepted, the function answers and is tested. The command doesn't
exist: `meow policy` has only `show`, so a user can't run
`meow policy explain` (from crates/meow-cli/src/surface.rs:300-303 and
crates/meow-cli/src/wire.rs:528-543, high). The pull request that built the
function left the command to M8 (from
https://github.com/retran/meowg1k/pull/119, high).

## Why

`meow policy explain` had promised the decision a real call would receive,
including one a person hasn't answered yet, which it can't know (from
https://github.com/retran/meowg1k/pull/111, high). A permission system whose
decisions can't be queried without triggering them is one people turn off (from
https://github.com/retran/meowg1k/pull/119, high). A grant is a fact about a run
in progress, and `explain` is a question asked before one (from
crates/meow-policy/src/prompt.rs:156-157, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: return the decision a real call would receive, including the answer to an `ask`, as the first draft of `R-POLICY-050` promised | One answer tells the user whether the call will run (from https://github.com/retran/meowg1k/pull/111, high) | That answer depends on a person who hasn't answered yet (from https://github.com/retran/meowg1k/pull/111, high) |
| No `explain`: learn a decision by making the call | Nothing to build, and the answer is the real outcome (reasoned from https://github.com/retran/meowg1k/pull/119, low) | A permission system whose decisions can't be queried without triggering them is one people turn off (from https://github.com/retran/meowg1k/pull/119, high) |
| Answer with the rule decision alone, ignoring grants (chosen) | The answer is a function of the policy and the call, so it is the same every time it is asked (from crates/meow-policy/src/prompt.rs:155-160, high) | It won |

## What it costs

`explain` can say `ask` for a call that a grant made during a run would allow,
so its answer differs from the outcome inside that run (from
crates/meow-policy/tests/spec.rs:573-583, high). The user has to know that
`ask` means "a person decides" and read it as no prediction (reasoned from
docs/spec/policy.md [R-POLICY-050], low).

## What would reverse it

- Grants outlive a process, so the answer to an `ask` becomes a stored fact
  `explain` could read; ADR-0107 keeps an always-grant out of storage, which is
  what holds this in place (reasoned from
  crates/meow-policy/src/prompt.rs:156-157, low).

## Consequences

- `Explanation` carries `decision`, `rule`, `origin` and `checked`, the fields
  `R-POLICY-051` asks the explanation to name (from
  crates/meow-policy/src/prompt.rs:135-146, high).
- `explain` passes `Grants::new()`, so no grant reaches it (from
  crates/meow-policy/src/prompt.rs:158, high).
- The command-line work is still owed: a `policy explain` subcommand that calls
  `meow_policy::explain` (from https://github.com/retran/meowg1k/pull/119,
  high).

## How I will know it was realised

1. `explain_predicts_the_rule_decision_without_making_the_call` in
   crates/meow-policy/tests/spec.rs passes: it names the rule, its origin and
   two checked rules, and a grant that makes `evaluate` allow a call leaves
   `explain` at `ask` (from crates/meow-policy/tests/spec.rs:538-584, high).
2. `meow policy explain <tool> <argument>` runs and prints the decision; this
   fails today, because the subcommand isn't declared (from
   crates/meow-cli/src/surface.rs:300-303, high).

## What this does not settle

- The command's output format and exit codes; no command exists yet (from
  crates/meow-cli/src/surface.rs:300-303, high).
- How a tool's arguments on the command line become a `Call` with the paths,
  commands or hosts a rule selects on (reasoned from
  crates/meow-star/src/run.rs:532-537, low).
