---
id: BUG-0123
artifact: bug
status: approved
severity: minor
violates: REQ-2442
found: 2026-09-27
revised: 2026-09-27
issue:
---

# No test checks that `search.text` and `search.files` obey the ignore files

The tests that name REQ-2442 all run in workspaces with no `.gitignore` and no
`.meowignore`, so none of them would fail if the two calls reported an ignored
file.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. List the tests that name REQ-2442:
   `grep -rn 2442 crates/*/tests`. They're in
   `crates/meow-cli/tests/search.rs:15`, `crates/meow-cli/tests/surface.rs:774`
   and `crates/meow-star/tests/running.rs:2429`, `:2465` and `:2495`.
2. Read each. None writes an ignore file.

The behaviour itself holds today: a workspace with `secret.txt` in
`.gitignore` and `m.txt` in `.meowignore` got `files hay.txt` from
`search.files("**/*.txt")`, and `search.text` reported neither ignored file.

## What the system does

The binary's `Searcher` walks with the index's `Walk`, which reads both ignore
files. No test pins that.

## What it should do, and why

REQ-2442: "`search.text` and `search.files` MUST obey the same walk the index
obeys, so a file the index ignores is a file they do not report." A test should
fail when either call reports an ignored file.

## Triage

The defect enters in the tests of `meow-cli`. It's minor: the behaviour is
right today and only the guard is missing. BUG-0030 is a different defect
against the same requirement, where a missing index model makes the walk fall
back to the default.

## Closed by

A test in `crates/meow-cli/tests/surface.rs`, named for example
`search_skips_what_the_ignore_files_exclude`, that writes one file excluded by
`.gitignore` and one by `.meowignore` and expects neither from `search.text` or
`search.files`.
