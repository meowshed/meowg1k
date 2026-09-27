---
id: BUG-0113
artifact: bug
status: approved
severity: minor
violates: REQ-1401
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A negation in a nested `.meowignore` can't re-include a git-ignored file

Only the root `.meowignore` gets the second walk that re-includes a file inside
a directory `.gitignore` excludes. The same negation in a `.meowignore` further
down the tree does nothing.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. In a scratch crate depending on `meow-index` by path, build a temporary tree
   with `.git/`, `.gitignore` holding `sub/generated/`, and
   `sub/generated/api.rs`.
2. Add `.meowignore` at the root holding `!sub/generated/api.rs` and run
   `Walk::default().run(root)`. `sub/generated/api.rs` is in `files`.
3. Instead add `sub/.meowignore` holding `!generated/api.rs` and run the walk
   again. `sub/generated/api.rs` is missing.

## What the system does

`Walk::reclaimed` reads only `root.join(".meowignore")`,
`crates/meow-index/src/walk.rs:173-176`, and the negations it finds are
resolved against the root. The first walk honours nested `.meowignore` files,
`walk.rs:125`, but under git's rule a negation can't reach inside an excluded
directory, which is why ADR-0117 added the second walk.

## What it should do, and why

REQ-1401: "`.meowignore` MUST support negation, so a path `.gitignore` excludes
can be indexed deliberately." A nested `.meowignore` is still a `.meowignore`,
and its negation should re-include the file as the root one does.

## Triage

The defect enters in `Walk::reclaimed`. It's minor: moving the line to the root
`.meowignore` works, and nested ignore files are less common.

## Closed by

A test in `crates/meow-index/tests/walk.rs`, named for example
`a_nested_meowignore_negation_reincludes_a_git_ignored_file`, running step 3 and
expecting the file in `files`.
