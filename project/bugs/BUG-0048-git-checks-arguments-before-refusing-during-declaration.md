---
id: BUG-0048
artifact: bug
status: approved
severity: minor
violates: REQ-2525
found: 2026-09-27
revised: 2026-09-27
issue:
---

# `@std//git` checks its arguments before refusing a call during declaration

`git.show("-x")` in `.meow/meow.star` fails with "`revision` may not begin with
`-`" in place of the declaration-phase refusal, because the argument check runs
before the phase check.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Create a workspace whose `.meow/meow.star` is `load("@std//git", "show")`
   followed by `show("-x")`.
2. Run `meow check` with `HOME` pointed at a scratch directory.

It prints "`revision` may not begin with `-`, because `git` would read `-x` as
an option" and exits 7. With `show("HEAD")` it prints "`git.show` is not
available while .meow/ is being loaded". Both were run against
`target/debug/meow`.

## What the system does

In `crates/meow-star/src/capability_git.rs`, `show` at lines 118-131 evaluates
`plain("revision", &revision)?` while building its argument list, and only then
calls `git`, which reaches `run_words` and the phase check. `diff` and `log` do
the same at lines 67-85 and 100-116. The steps in `CLAUDE.md` ask every module to
call `running(eval, ...)` first.

## What it should do, and why

REQ-2525: "While `.meow/` is being evaluated, every other runtime module MUST
fail with an error saying it is unavailable during declaration." ADR-0420
records the decision. No test checks `@std//git`'s refusal during declaration.

## Triage

The defect enters in `meow-star`'s `git` module. It is minor: the call is
refused either way and only the message is wrong. The fix is to call
`running(eval, "git.<call>")` at the top of each function.

## Closed by

A test in `crates/meow-star/tests/running.rs`, named for example
`git_is_refused_during_declaration_before_its_arguments_are_checked`, that
expects the phase refusal for `show("-x")`.
