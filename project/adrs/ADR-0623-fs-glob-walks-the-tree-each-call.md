---
id: ADR-0623
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2550, REQ-2551, REQ-2552]
supersedes: []
---

# 0623. `fs.glob` walks the whole workspace on each call and skips `.data`

## Decision

`fs.glob(pattern)` walks every directory under the workspace root on each call,
skips any directory named `.data`, matches each file's path relative to the
root, written with `/`, against the pattern and returns the matches sorted
(from https://github.com/meowshed/meowg1k/pull/135 and
crates/meow-star/src/capability.rs:142-185, high). An invalid pattern fails
with the pattern named (from crates/meow-star/src/capability.rs:150-152, high).

Once this is accepted, it works as stated; it has no cache and prunes nothing
beyond `.data` (from https://github.com/meowshed/meowg1k/pull/135, high).

## Why

The store is never walked, because globbing it or indexing it both end with a
workspace describing itself (from crates/meow-star/src/capability.rs:162-163,
high). The order is stable so a handler that writes its results to a file
produces the same file twice (from crates/meow-star/src/capability.rs:180-181,
high). A full walk is fine for a repository, and the workspace boundary already
keeps it out of a home directory (from
https://github.com/meowshed/meowg1k/pull/135, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Cache the walk between calls | A handler that globs in a loop doesn't walk the tree each time (from https://github.com/meowshed/meowg1k/pull/135, high) | Not needed for a repository-sized tree, and a cache has to know when files change (reasoned from https://github.com/meowshed/meowg1k/pull/135, low) |
| Walk as the index does, obeying `.gitignore` and `.meowignore`, as `search.files` does | Skips `target/` and `.git/`, and agrees with `search.files` about what exists (reasoned from docs/requirements/REQ-2442-search-obeys-index-walk.md, low) | Not taken in #135, which names "no early pruning beyond skipping `.data`" as a known limit (from https://github.com/meowshed/meowg1k/pull/135, high) |

Doing nothing isn't an option, because `fs.glob` was new in #135 (from
https://github.com/meowshed/meowg1k/pull/135, high).

## What it costs

Each call reads every directory in the workspace, `.git/` and build output
included, which is wrong for a very large tree (from
https://github.com/meowshed/meowg1k/pull/135, high). The walk follows a linked
directory, so a link to an ancestor would walk without end (reasoned from
crates/meow-star/src/capability.rs:166, low). A directory named `.data`
anywhere is skipped, not only `.meow/.data/` (from
crates/meow-star/src/capability.rs:164, high).

## What would reverse it

- A workspace large enough that `fs.glob` is slow in a handler, or a report of
  a walk that doesn't end (reasoned from
  crates/meow-star/src/capability.rs:155-178, low).

## Consequences

- `fs.glob` and `search.files` can disagree about which files exist, because
  only `search.files` obeys the ignore files (reasoned from
  docs/requirements/REQ-2442-search-obeys-index-walk.md, low).
- Paths come back with `/`, as ADR-2410 asks (from
  crates/meow-star/src/capability.rs:175 and ADR-2410, high).

## How I will know it was realised

1. `fs_reads_and_writes_inside_the_workspace` in
   crates/meow-star/tests/running.rs passes; it globs `notes/**/*.txt` and
   expects the matches in order (from crates/meow-star/tests/running.rs:804-840,
   high). No test covers the `.data` skip or an invalid pattern.

## What this does not settle

- Whether `fs.glob` should obey `.gitignore` like the search module (reasoned
  from docs/requirements/REQ-2442-search-obeys-index-walk.md, low).
