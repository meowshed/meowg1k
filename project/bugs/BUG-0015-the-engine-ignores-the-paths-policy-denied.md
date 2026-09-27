---
id: BUG-0015
artifact: bug
status: approved
severity: major
violates: REQ-2013
found: 2026-09-27
revised: 2026-09-27
issue:
---

# The engine ignores the paths a policy verdict denied

`Policy::evaluate` returns `Allow` together with `denied_paths` when a rule
covers some of a call's paths and misses others. The engine reads only the
decision, so a write whose paths include one no rule allows runs whole, and a
read runs on every path without the model being told which it may not see.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare a workspace policy that allows tool `save` on paths `src/**`, and a
   tool `save` whose arguments include `paths`, a list, and whose handler writes
   each path.
2. Run an agent against a recorded model reply that calls `save` with `paths =
   ["src/a.rs", ".git/config"]`.

`.git/config` is written. The call should be refused as a whole.

## What the system does

`crates/meow-policy/src/policy.rs:308-318` splits the paths into matched and
missed and returns `Allow` with the missed ones in `denied_paths`. The engine at
`crates/meow-agent/src/engine.rs:358-395` uses `verdict.decision` and
`verdict.rule` and never reads `denied_paths`; `grep -rn denied_paths crates`
finds no reader outside `meow-policy`. The tests at
`crates/meow-policy/tests/spec.rs:155-190` check the verdict's fields and pass
without any caller acting on them: the write test asserts only that
`call.access` is `Write`.

## What it should do, and why

REQ-2013: "A write that hits a denied path MUST be denied as a whole." REQ-2014
adds that such a write MUST NOT write any path. For a read, REQ-2011 and
REQ-2012 ask the call to return the allowed paths and name every skipped one.

## Triage

The defect enters in `meow-agent`'s `invoke`, which drops half the verdict. It
is major because the requirement's main behaviour doesn't happen and a write
reaches a path no rule allows. It needs a tool that takes several paths, which
no built-in module offers as a tool today. The fix is for the engine to deny a
write with any denied path and to hand a read's tool only the allowed paths,
with the skipped ones named in the result.

## Closed by

A test in `crates/meow-agent/tests/`, named for example
`a_write_with_one_denied_path_runs_nothing`, and one named
`a_read_with_one_denied_path_names_what_it_skipped`, both driving the engine
rather than `Policy::evaluate` alone.
