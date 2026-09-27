---
id: ADR-0463
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1807, REQ-1808]
supersedes: []
---

# 0463. A package's tree hash is recomputed on every load

## Decision

The tree is hashed over sorted relative paths and the bytes at each, recomputed
rather than read from a marker file, and compared against the lockfile before
anything is evaluated (from https://github.com/retran/meowg1k/pull/157, high).

Once this is accepted, `package::resolve` hashes the cached tree with
`hash_tree` on every `@<pkg>//` load and refuses a mismatch with a message that
names the package, the expected hash and the one found, before the file is
evaluated (from crates/meow-star/src/package.rs:199, high). `meow pkg fetch`
uses the same `hash_tree` on the staged download, so fetching and loading can't
disagree about what a hash covers (from crates/meow-cli/src/fetch.rs:154, high).
The file is read again when it is evaluated, after the hash was taken, so a file
changed between the two reads isn't caught (reasoned from
crates/meow-star/src/loader.rs:129, low).

## Why

A marker is written by the same process that would have been fooled, so a cache
somebody edited, or a mirror that served something else, would still run (from
https://github.com/retran/meowg1k/pull/157, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Read the hash from a marker file written at fetch time | A load reads one small file and hashes nothing, so its cost doesn't grow with the package (reasoned from crates/meow-star/src/package.rs:116, low) | The process that writes the marker is the one that would have been fooled (from https://github.com/retran/meowg1k/pull/157, high) |
| Cache the computed hash against modification times | A large package isn't re-read on every load (from https://github.com/retran/meowg1k/pull/157, high) | Packages are kilobytes of Starlark and the hash takes microseconds, so the pull request shipped the plain version and left the optimisation out (from https://github.com/retran/meowg1k/pull/157, high) |
| Do nothing: trust the cache as it stands, since it is keyed by hash | No hashing at load at all (reasoned from crates/meow-star/src/package.rs:99, low) | A loader that verified nothing would pass every other test, and REQ-1807 requires the check (from https://github.com/retran/meowg1k/pull/157, high) |

## What it costs

The hash is recomputed on every load; packages are kilobytes of Starlark, and a
large package would want the result cached against modification times (from
https://github.com/retran/meowg1k/pull/157, high). `resolve` runs before the
loader looks in its evaluate-once cache, so each `load` of a package hashes the
whole tree again, even when the module was already evaluated in this run (from
crates/meow-star/src/loader.rs:129 and crates/meow-star/src/loader.rs:146,
high).

## What would reverse it

A package large enough that hashing it on every load shows in a run's start-up
time would call for caching the result against modification times, as the pull
request names (from https://github.com/retran/meowg1k/pull/157, high). A
verified signature on the cache entry, which a user-edited cache couldn't
forge, would also remove the need to recompute (reasoned from
crates/meow-star/src/package.rs:99, low).

## Consequences

- The cache is keyed by the hash and verified by the hash, so two workspaces
  that pinned one hash run the same bytes (from
  crates/meow-star/src/package.rs:95, high).
- Moving a file inside a package changes its hash, because the hash covers the
  sorted relative paths as well as the bytes (from
  crates/meow-star/src/package.rs:106, high).
- A mismatch tells the user to run `meow pkg fetch`, which replaces the cache
  entry (from crates/meow-star/src/package.rs:204, high).

## How I will know it was realised

Short-circuiting the comparison fails
`contents_that_do_not_match_the_lockfile_are_refused`, which edits a cached
package after locking it (from https://github.com/retran/meowg1k/pull/157,
high). That test and `moving_a_file_changes_the_hash` are in
`crates/meow-star/tests/packages.rs` (from
crates/meow-star/tests/packages.rs:110, high).

## What this does not settle

- A file changed after the hash is taken and before it is evaluated isn't
  caught (reasoned from crates/meow-star/src/loader.rs:129, low).
- `collect` follows a symbolic link to a directory, because `Path::is_dir`
  follows links, so a link inside the cache entry is hashed as the tree it
  points at (reasoned from crates/meow-star/src/package.rs:143, low).
