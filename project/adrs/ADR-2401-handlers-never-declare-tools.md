---
id: ADR-2401
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2475, REQ-2476]
supersedes: []
---

# 2401. A handler may not declare a tool

## Decision

`meow.provider`, `meow.model`, `meow.agent`, `meow.tool`, `meow.command` and
`meow.policy` are callable only while `.meow/` is being evaluated, and fail
inside a handler (from docs/spec/starlark.md [R-STAR-030], high).

## Why

Generated tools would make the tool set unknowable before a run, which breaks
`meow policy explain` and with it the promise that a permission decision can be
predicted without triggering it (from docs/spec/starlark.md, Decisions, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Let a handler generate tools at run time | Generated tools would be useful (from docs/spec/starlark.md, Decisions, high). | The tool set becomes unknowable before a run, which breaks `meow policy explain` (from docs/spec/starlark.md, Decisions, high). |
| Do nothing: leave the question open, as the draft spec did | A workspace author could decide per case (reasoned from git show e39cd2f -- docs/spec/starlark.md, low). | The draft listed it as an open question and recommended forbidding it for the same reason, and e39cd2f closed it that way (from git show e39cd2f -- docs/spec/starlark.md, high). |
| Accept `meow.policy` inside a handler and apply it from that point on | A handler could tighten its own permissions mid-run (reasoned from crates/meow-star/src/declare.rs `policy`, low). | A policy written inside a handler would be applied after the calls it was meant to govern, which reads as a permission bug (from crates/meow-star/src/declare.rs `policy` and https://github.com/retran/meowg1k/pull/121, high). |

## What it costs

A workspace can't generate tools, which the source calls useful (from
docs/spec/starlark.md, Decisions, high). A tool set that depends on the
repository, for example one tool per package, has to be written out by hand in
`.meow/` (reasoned from crates/meow-star/src/declare.rs `declaring`, low).

## What would reverse it

A way to know every tool a handler could declare before the run starts would
remove the reason, because the objection is that the tool set becomes
unknowable before a run (reasoned from docs/spec/starlark.md, Decisions, low).

## Consequences

`meow policy explain` can predict a permission decision without triggering it,
because the tool set is known before a run (from docs/spec/starlark.md,
Decisions, high). The trust fingerprint is taken over the declared agent, tool,
command and policy lines, so a tool that appeared only at run time would escape
what the person agreed to (from https://github.com/retran/meowg1k/pull/154,
high). The refusal names the phase: `<call> can only be called while .meow/ is
being loaded` (from crates/meow-star/src/error.rs `NotDeclaring`, high).

## How I will know it was realised

`a_handler_may_not_declare_anything` in crates/meow-star/tests/running.rs calls
`meow.provider` from a handler, expects the refusal, and checks the registry
did not gain the provider (from that test, high). The test covers
`meow.provider` only; the other five calls go through the same `declaring`
check, as do `meow.agent_named`, `meow.package` and `meow.index` (from
crates/meow-star/src/declare.rs lines 163 to 403, high).

## What this does not settle

Whether `meow policy explain` exists in the binary: `meow-policy` has `explain`,
but `meow policy --help` lists only `show` (from
crates/meow-policy/src/prompt.rs `explain` and `target/debug/meow policy
--help` run on 2026-09-27, high). The decision's reason rests on a command the
binary doesn't expose yet.
