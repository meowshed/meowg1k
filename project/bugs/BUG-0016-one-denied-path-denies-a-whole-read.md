---
id: BUG-0016
artifact: bug
status: approved
severity: minor
violates: REQ-2011
found: 2026-09-27
revised: 2026-09-27
issue: 189
---

# An explicit deny on one path denies a whole read

When a deny rule matches one of a read's paths, `Policy::evaluate` returns
`Deny` for the call. A read that touches one denied path among several allowed
ones is refused outright, where REQ-2011 asks it to return the allowed paths.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Build a `Policy` that denies `fs.glob` on `/w/.env` and allows `fs.glob` on
   `/w/**`.
2. Evaluate `Call::new("fs.glob", Access::Read)` with paths `/w/src/a.rs` and
   `/w/.env`.

The verdict is `Deny`, with `denied_paths` holding only `/w/src/a.rs`, the path
the deny rule missed.

## What the system does

`crates/meow-policy/src/policy.rs:251-293` walks deny rules first and returns
the first rule whose selector hits any path. `selector_hit` at
`crates/meow-policy/src/policy.rs:308-318` counts a rule as hit when at least one
path matches, so a deny rule matching one path decides the whole call, and
`denied_paths` then lists the paths the deny rule did not match, the reverse of
its meaning under an allow.

## What it should do, and why

REQ-2011: "A read that hits a denied path MUST return the allowed paths."
REQ-2012 adds that it names every path it skipped. REQ-2010 judges a multi-path
call once per path, which the per-call deny doesn't do.

## Triage

The defect enters in `meow-policy`'s `evaluate`. It is minor: it refuses more
than it should, so no boundary is crossed, and BUG-0015 means the engine
wouldn't act on a per-path answer yet. The fix is to judge each path of a read
separately and return the allowed set with the denied set.

## Closed by

A test in `crates/meow-policy/tests/spec.rs`, named for example
`an_explicit_deny_on_one_path_leaves_the_rest_of_a_read`, that runs the steps
above and expects `Allow` with `/w/.env` in `denied_paths`.
