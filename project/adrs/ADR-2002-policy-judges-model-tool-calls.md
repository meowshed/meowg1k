---
id: ADR-2002
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2037, REQ-2038]
supersedes: []
---

# 2002. Policy governs what a model decided, not what a script did

## Decision

The engine evaluates the policy for a tool call a model makes, and a handler
calling a capability directly isn't judged (from docs/spec/policy.md
[R-POLICY-040], high). A handler calling `fs.write` through `load("@std//fs",
...)` runs as code; the same function offered to a model in a `tools` list is a
tool call, goes through the engine and is judged (from docs/spec/policy.md,
high).

Once this is accepted, the one call to `Policy::evaluate` on the run path is
in the engine's `invoke`, and no capability module under `crates/meow-star/src`
imports `meow_policy` (from crates/meow-agent/src/engine.rs:357-360, and
`grep -l meow_policy crates/meow-star/src/*.rs`, which lists only `agent.rs`,
`registry.rs` and `run.rs`, high). A handler's own `http`, `fs` and `shell`
calls run with no prompt (from crates/meow-star/src/capability_http.rs:19-23,
high).

## Why

A handler is code the workspace's own author wrote, and asking them to approve
their own script is a prompt nobody reads (from docs/spec/policy.md, high).
Enforcing in the engine alone gives the boundary one enforcement point, where
two would be two places for a decision to differ, which is the defect the policy
layer exists to prevent (from docs/spec/policy.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Judge a script's own capability calls as well | One rule would cover a path whether a model or a handler touches it (reasoned from docs/spec/policy.md, Decisions, low) | Asking the workspace's author to approve their own script is a prompt nobody reads (from docs/spec/policy.md, high), and it puts a prompt in front of the author's own program (from https://github.com/meowshed/meowg1k/pull/145, high) |
| Enforce in the engine and inside each capability module | A capability would refuse a denied call even on a path that skipped the engine (reasoned from docs/spec/policy.md, Decisions, low) | Two enforcement points are two places for a decision to differ, which is the defect the policy layer exists to prevent (from docs/spec/policy.md, high) |

Doing nothing isn't an option here: v0.2.x had no policy layer, and the choice
to have one is ADR-2004 (from docs/spec/policy.md, Changes from v0.2.x, high).

## What it costs

A handler that writes a file or runs a command does so with no rule and no
prompt, so a workspace author who gives a model a tool taking a URL or a path
has handed the model what the tool reaches, and the rule for that tool is the
only place to narrow it (from https://github.com/meowshed/meowg1k/pull/145 and
docs/design/0.3.0-starlark-api.md section 10, high). Code from a package runs
as a handler too, and it isn't code the workspace's author wrote (reasoned from
docs/spec/packages.md [R-PKG-033], low).

## What would reverse it

- A handler's direct capability call shown to cause harm that no rule on a
  model's tool call could have stopped, for example package code writing
  outside what its tool's rule allows, would reopen the question (reasoned
  from docs/spec/packages.md [R-PKG-033], low).

## Consequences

The boundary has one enforcement point, in the engine (from docs/spec/policy.md,
high). A tool that wants to be governed names its arguments `path`, `paths`,
`file`, `files`, `command` or `url`, because the runtime describes a call to the
policy by those names (from crates/meow-star/src/run.rs:515-566, high).

## How I will know it was realised

1. `policy_stops_a_tool_before_it_runs_and_says_so` in
   crates/meow-agent/tests/spec.rs passes: a denied tool never runs and the
   decision is recorded (from crates/meow-agent/tests/spec.rs:761-801, high).
2. `grep -l meow_policy crates/meow-star/src/capability*.rs` finds nothing, so
   no capability module judges its own calls (from the grep over
   crates/meow-star/src, high).

## What this does not settle

- Whether code from a package, which runs as a handler, should be judged, since
  the reason given for exempting a handler is that the workspace's author wrote
  it (reasoned from docs/spec/packages.md [R-PKG-033] and docs/spec/policy.md,
  Decisions, low).
