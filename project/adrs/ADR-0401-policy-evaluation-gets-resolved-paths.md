---
id: ADR-0401
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2004, REQ-2005, REQ-2020, REQ-2021]
supersedes: []
---

# 0401. Policy evaluation receives resolved paths and touches nothing

## Decision

The caller resolves paths before evaluation, so evaluation itself touches no
filesystem (from https://github.com/meowshed/meowg1k/pull/110, high). Grants are
an argument to evaluation rather than ambient state, so the answer is a function
of its inputs (from https://github.com/meowshed/meowg1k/pull/119, high).

Once this is accepted, `Policy::evaluate` takes a `Call` whose paths are
already resolved and a `Grants` value, and `meow_policy::resolve` sits outside
it for the caller to use (from crates/meow-policy/src/policy.rs:244-251 and
crates/meow-policy/src/policy.rs:413-421, high). The runtime's `describe_call`
resolves each path argument before the engine asks for a decision (from
crates/meow-star/src/run.rs:517-582, high). What still doesn't work is handing
the resolved path to the tool: the engine passes the model's own arguments to
the tool, so the tool acts on the path as the model wrote it (from
crates/meow-agent/src/engine.rs:355-408, medium).

## Why

`R-POLICY-013` required evaluation free of input and output while `R-POLICY-003`
had path matching resolve symlinks, and the two couldn't both hold (from
https://github.com/meowshed/meowg1k/pull/110, high). Evaluation also had to be a
pure function of the call and the policy while session grants changed its answer
(from https://github.com/meowshed/meowg1k/pull/111, high). Paths that arrive
resolved, with the tool acting on exactly those, close the window in which a
path allowed as a file becomes a symlink to somewhere denied (from
https://github.com/meowshed/meowg1k/pull/119, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: path matching resolves symlinks during evaluation, as `R-POLICY-003` first read | Every caller gets resolution for free, and no caller can forget to resolve (reasoned from crates/meow-policy/src/rule.rs:133-140, low) | It contradicts the requirement that evaluation performs no input or output (from https://github.com/meowshed/meowg1k/pull/110, high), and it opens a window between the check and the use (from crates/meow-policy/src/rule.rs:133-140, high) |
| Session grants held as state the evaluator reads | The caller passes nothing extra, and a grant made anywhere applies everywhere at once (reasoned from crates/meow-policy/src/policy.rs:160-171, low) | Grants change the answer, so evaluation stops being a function of the call and the policy (from https://github.com/meowshed/meowg1k/pull/111, high) |

The history names these two options and no third; the question has two
independent parts, paths and grants, and each part had one rejected option
(from https://github.com/meowshed/meowg1k/pull/110 and
https://github.com/meowshed/meowg1k/pull/111, high).

## What it costs

A path that doesn't exist yet can't be resolved, so a write to a new file is
judged on its joined absolute path without symlink resolution, and that is the
one place the guarantee of `R-POLICY-003` is weaker than it reads (from
https://github.com/meowshed/meowg1k/pull/127, high). Every caller also carries
the duty to resolve: the runtime does it by argument name, so an argument not
named `path`, `paths`, `file` or `files` is never judged as a path (from
crates/meow-star/src/run.rs:521-525 and crates/meow-star/src/run.rs:537-551,
high).

## What would reverse it

- Evaluation needs to read something only the filesystem holds at decision
  time, such as a file's owner or mode, which a resolved path alone can't give
  it (reasoned from crates/meow-policy/src/policy.rs:78-96, low).

## Consequences

- `Rule::matches_path` matches the path it is given and never resolves it (from
  crates/meow-policy/src/rule.rs:133-140, high).
- The engine takes a `describe_call` from its caller, and until it was
  supplied no decision was ever taken: `describe_call` was `None` before the
  approval change (from https://github.com/meowshed/meowg1k/pull/127, high).
- The runtime starts each run with empty grants, and an "always" answer is
  remembered for the process and written nowhere (from
  crates/meow-star/src/run.rs:366 and
  https://github.com/meowshed/meowg1k/pull/127, high).

## How I will know it was realised

1. `evaluation_is_the_same_answer_every_time` in
   crates/meow-policy/tests/spec.rs passes: the same call and policy give `ask`
   with no grants and `allow` with a grant, three times over (from
   crates/meow-policy/tests/spec.rs:266-280, high).
2. `a_path_selector_crosses_directories_with_a_double_star` in
   crates/meow-policy/tests/spec.rs passes on absolute paths the test supplies
   already resolved (from crates/meow-policy/tests/spec.rs:96-100, high).

## What this does not settle

- Whether the tool acts on the resolved path. REQ-2006 and REQ-2007 require
  it, the only test checks the shape of a `Call` and says it can check no more,
  and the engine calls the tool with the model's arguments (from
  crates/meow-policy/tests/spec.rs:282-287 and
  crates/meow-agent/src/engine.rs:355-408, medium).
- How a declaration marks an argument as a path: it can't yet, and the history
  says that is worth a requirement and has none (from
  https://github.com/meowshed/meowg1k/pull/127, high).
