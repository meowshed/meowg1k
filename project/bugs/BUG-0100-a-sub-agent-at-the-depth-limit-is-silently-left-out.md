---
id: BUG-0100
artifact: bug
status: approved
severity: major
violates: REQ-1052
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A sub-agent at the depth limit is silently left out of its caller's tools

When `meow-star` builds the tool set for an agent at depth 2, it drops every
sub-agent that agent names, so the model is never offered the sub-agent and
never gets the tool error REQ-1052 asks for.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare three markdown agents, `a`, `b` and `c`, where `a` lists `b` in
   `tools`, `b` lists `c`, and `c` lists `a`.
2. Build the spec for `a` through `Runtime::agent_spec("a", 0)`.
3. Inspect the tool set of `c`'s spec, at depth 2. It holds no tool named `a`.

This follows from the code below; no test in `crates/meow-star/tests` covers
nesting depth.

## What the system does

`crates/meow-star/src/run.rs:408-411` skips a sub-agent with `continue` when
`depth + 1 >= MAX_DEPTH`, and `MAX_DEPTH` is 3 at
`crates/meow-star/src/run.rs:33`. The engine's own refusal in
`SubAgent::call`, `crates/meow-agent/src/nested.rs:78-86`, which returns the
tool error, is never reached from a Starlark workspace. The comment on
`MAX_DEPTH` says the limit holds "unless a declaration says otherwise", and no
declaration can change it.

## What it should do, and why

REQ-1052: "The engine MUST report a refusal to start a sub-agent beyond the
maximum nesting depth as a tool error rather than a panic." The model at depth 2
should see its declared sub-agent, call it, and get a tool error saying the
nesting limit was reached, so it can choose another approach.

## Triage

The defect enters in `Runtime::tools_for`, which bounds spec construction by
leaving tools out. ADR-0423 records this as a cost of building specs eagerly.
It's major because the requirement's behaviour, a refusal the model is told
about, doesn't happen on the only path a user has. The fix is to offer the
sub-agent at the limit as a tool whose call returns the refusal, or to build
sub-agent specs lazily as ADR-0423's reversal condition describes.

## Closed by

A test in `crates/meow-star/tests/`, named for example
`a_sub_agent_past_the_limit_is_refused_as_a_tool_error`, that declares two
agents naming each other, drives a scripted model that calls the sub-agent at
depth 2, and expects a tool result naming the limit.
