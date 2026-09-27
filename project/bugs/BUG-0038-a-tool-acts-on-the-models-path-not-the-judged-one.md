---
id: BUG-0038
artifact: bug
status: approved
severity: major
violates: REQ-2006
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A tool acts on the model's raw path, not the path the policy judged

The engine resolves a call's paths, with symbolic links followed, only to judge
them. It then hands the tool the model's original arguments, and the tool
resolves the path a second time when it opens the file. Between the two, a path
allowed as a plain file can become a link to a denied one.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare a policy that allows tool `read` on `src/**` and a tool `read` whose
   handler returns the contents of its `path` argument.
2. Run an agent against a recorded reply calling `read` with `path =
   "src/a.txt"`, where `src/a.txt` is a plain file when the policy judges it.
3. Between the verdict and the handler's read, replace `src/a.txt` with a link
   to `../.env`, for example from a test approver that swaps the file before
   answering.

The handler reads `.env`. This was traced through the code and not run.

## What the system does

`describe_call` in `crates/meow-star/src/run.rs:525-568` resolves each path
through `meow_policy::resolve`, which is `std::fs::canonicalize`
(`crates/meow-policy/src/policy.rs:419-421`). `Engine::invoke` in
`crates/meow-agent/src/engine.rs:338-419` judges that `Call` and then calls
`tool.call(&args, ...)` with `args` parsed from the model's raw arguments. The
test `the_judged_path_is_the_one_to_act_on` at
`crates/meow-policy/tests/spec.rs:282-299` checks only that evaluation leaves a
`Call` unchanged, and its comment says it can check nothing more.

## What it should do, and why

REQ-2006: "A tool MUST act on the exact path the policy evaluated." REQ-2007: "A
tool MUST NOT resolve the path a second time. Re-resolving reopens the window in
which a path allowed as a file becomes a symlink to somewhere denied." ADR-0401
records the decision.

## Triage

The defect enters in `meow-agent`'s `invoke`, which judges one value and runs
another. It is major because the requirement's main behaviour doesn't happen
and a race gets past the path policy. Exploiting it needs something that can
change the workspace during a run, such as another tool the same agent calls, so
it isn't rated critical. The fix is to rewrite the path arguments to the judged,
resolved paths before `tool.call`.

## Closed by

A test in `crates/meow-agent/tests/`, named for example
`a_tool_receives_the_resolved_path_the_policy_judged`, that records the
arguments the tool received and expects the canonical path, not the model's.
