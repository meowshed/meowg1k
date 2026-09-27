---
id: BUG-0032
artifact: bug
status: approved
severity: minor
violates: REQ-2474
found: 2026-09-27
revised: 2026-09-27
issue:
---

# `path.base` returns a backslash on Unix

On Unix, `path.base("a\\b\\c.rs")` returns `a\b\c.rs`, the whole string with its
backslashes, because `\` isn't a separator there. REQ-2474 says no `path`
function returns `\`.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Create a workspace whose command handler writes `base("a\\b\\c.rs")`, with
   `base` loaded from `@std//path`.
2. Run `meow trust`, then the command, on macOS.

It prints `base=a\b\c.rs`. This was run against `target/debug/meow`.

## What the system does

`path.base` at `crates/meow-star/src/modules.rs:694-704` takes the file name of
the native path. Pull request #149, "write one path separator, and make it a
slash", chose to treat `\` as an ordinary character on Unix, so a file whose
name contains `\` keeps its name.

## What it should do, and why

REQ-2474: "`path` MUST NOT return `\`, so that a path `path.join` built and a
path `search.files` reported can be compared, and a `.meow/` written once
behaves the same everywhere."

## Triage

The defect enters where REQ-2474 and the choice in pull request #149 disagree.
The requirement is the part that is probably wrong: on Unix `\` is a legal
filename character, and rewriting it would rename a real file. Amending REQ-2474
to "`path` MUST NOT return `\` where `\` is a separator" would close this. It is
minor because it needs a backslash in a Unix path, which no tool the workspace
calls produces.

## Closed by

A test in `crates/meow-star/tests/running.rs` citing the amended REQ-2474, named
for example `path_base_keeps_a_backslash_that_is_not_a_separator`.
