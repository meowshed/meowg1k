---
id: BUG-0031
artifact: bug
status: approved
severity: major
violates: REQ-2046
found: 2026-09-27
revised: 2026-09-27
issue:
---

# `meow policy explain` doesn't exist

The `policy` group offers only `show`. `meow policy explain <tool> <argument>`
is a usage error, so a user can't ask what a call would get before an agent
makes it.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run `meow policy explain shell "rm -rf build"` in any workspace.

It prints `error: unrecognized subcommand 'explain'` and exits 2. This was run
against `target/debug/meow`.

## What the system does

`crates/meow-cli/src/surface.rs:300-303` declares `policy` with the single
subcommand `show`, and `crates/meow-cli/src/wire.rs:528-543` handles `show` and
maps anything else to `Ending::Usage`. `meow_policy::explain` exists
(`crates/meow-policy/src/lib.rs:28`) and nothing in `meow-cli` calls it.

## What it should do, and why

REQ-2046: "`meow policy explain <tool> <argument>` MUST return the rule decision
a real call with those arguments would receive: `allow`, `ask`, or `deny`."

## Triage

The defect enters in `meow-cli`'s command surface. It is major because the
requirement's whole behaviour is missing. No open issue tracks it. Its answer
should come from the same `describe_call` and `evaluate` the engine uses, or the
explanation and the real decision will drift apart.

## Closed by

A test in `crates/meow-cli/tests/`, named for example
`policy_explain_prints_the_decision_a_real_call_would_get`, that declares a
policy and expects `deny`, `ask` and `allow` for three calls.
