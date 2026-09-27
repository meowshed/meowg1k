---
id: ADR-0117
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1401]
supersedes: []
---

# 0117. `.meowignore` negations are walked separately from the ignore set

## Decision

The walker handles `.meowignore` negations in a second walk over the named
paths, apart from the merged ignore set (from docs/design/0.3.0-plan.md M10,
high). The second walk turns every ignore file off, takes the negations as its
only filter, holds each file it finds to the same size, binary and escape rules
as the first walk, and merges the result in (from
crates/meow-index/src/walk.rs:162-228, high).

## Why

Git's rule is that a file inside an excluded directory can't be re-included, so
`!generated/api.rs` alone does nothing when `.gitignore` holds `generated/`
(from docs/design/0.3.0-plan.md M10, high). [R-INDEX-001] asks for one line to
be enough, and the second walk is what makes it enough (from
docs/design/0.3.0-plan.md M10, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: merge the negations into one ignore set, under git's rules | One walk, and `.meowignore` means exactly what the same line means in `.gitignore` (reasoned from crates/meow-index/src/walk.rs:110-125, low) | Under git's rule a negation can't re-include a file inside an excluded directory, because the walk never descends far enough to see it (from crates/meow-index/src/walk.rs:162-172, high) |
| Ask the user to re-include each parent directory first, as git requires | No second walk, and git's semantics hold unchanged (reasoned from docs/design/0.3.0-plan.md M10, low) | [R-INDEX-001] asks for one line to be enough (from docs/design/0.3.0-plan.md M10, high) |

The plan, the code and the pull request weigh no third way to re-include a file
(from docs/design/0.3.0-plan.md M10 and
https://github.com/retran/meowg1k/pull/130, high).

## What it costs

A `.meowignore` with negations walks the tree twice (from
crates/meow-index/src/walk.rs:108-110 and 201-211, high). Only the root
`.meowignore` is read for negations, so a negation in a nested `.meowignore`
gets no second walk (from crates/meow-index/src/walk.rs:173-176, high).

## What would reverse it

- The `ignore` crate gaining a mode in which a later negation re-includes a
  file inside a directory an earlier file excluded, which would make one walk
  enough (reasoned from crates/meow-index/src/walk.rs:110-125, low).

## Consequences

`.meow/.data/` is excluded in both walks, so no negation brings the store back
into the index (from crates/meow-index/src/walk.rs:93-102 and 219-221, high).
A file that arrives through a negation is held to the same size, binary and
escape rules as one that arrives normally (from
crates/meow-index/src/walk.rs:230-234, high).

## How I will know it was realised

1. `meowignore_can_re_include_what_git_excludes` in
   crates/meow-index/tests/walk.rs passes: `!generated/api.rs` brings back that
   file and no other (from crates/meow-index/tests/walk.rs:59, high).
2. `the_data_directory_cannot_be_re_included` in the same file passes (from
   crates/meow-index/tests/walk.rs:80, high).

## What this does not settle

- Negations in a `.meowignore` below the root, which the second walk doesn't
  read (from crates/meow-index/src/walk.rs:173-176, high).
- Which files a walk skips for their size, for binary content or for a link
  that leaves the workspace, which REQ-1403 to REQ-1407 cover (from
  crates/meow-index/src/walk.rs:77-83, high).
