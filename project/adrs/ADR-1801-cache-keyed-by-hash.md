---
id: ADR-1801
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1816]
supersedes: []
---

# 1801. The package cache is keyed by hash and not by name and version

## Decision

The package cache is keyed by the hash of a package's contents (from
docs/spec/packages.md [R-PKG-021], high). The cache entry is
`.meow/.data/pkg/<sha256>` (from crates/meow-star/src/package.rs:96-103, high).

Once this is accepted, two packages with the same contents share one entry and
a changed pin can't be served the old bytes (from crates/meow-cli/tests/fetch.rs
`identical_contents_share_a_cache_entry`, high). A person can't tell which
package an entry holds from its directory name alone, and reads `meow pkg list`
for that (reasoned from crates/meow-star/src/package.rs:96-103, low).

## Why

A version is a label upstream controls and can move, and a hash is the bytes.
Two workspaces that pinned the same hash are provably running the same code,
which is the property the lockfile exists to give (from docs/spec/packages.md,
Decisions, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Key the cache by name and version | A person can read which package and version an entry holds from its path (reasoned from crates/meow-star/src/package.rs:96-103, low). | A version is a label upstream controls and can move (from docs/spec/packages.md, Decisions, high). |
| Do nothing: keep no cache and fetch on every run | No cache to verify or clean (reasoned from docs/spec/packages.md [R-PKG-020], low). | A workspace that has fetched once must work offline, and a load must never fetch (from docs/spec/packages.md [R-PKG-012] [R-PKG-020], high). |

Neither the source nor the history names a third key for the cache (from
docs/spec/packages.md and https://github.com/retran/meowg1k/pull/157, medium).

## What it costs

The loader hashes the whole cached tree on every load, sorted paths and bytes,
and doesn't trust a stamp file (from crates/meow-star/src/package.rs:105-133,
high). That is kilobytes and microseconds for Starlark files, and a large
package would want the result cached against modification times (from
https://github.com/retran/meowg1k/pull/157, high).

## What would reverse it

- Upstream versions that are immutable by construction, so that a name and a
  version identify the bytes as well as the hash does, would remove the reason
  the source gives (reasoned from docs/spec/packages.md, Decisions, low).

## Consequences

Two workspaces pinning the same package share one copy, and a changed pin can't
be served the old bytes (from docs/spec/packages.md [R-PKG-021], high). A
package already cached under its hash isn't downloaded again (from
https://github.com/retran/meowg1k/pull/162, high). Moving a file inside a
package changes its hash, because the hash covers the paths as well as the
bytes (from crates/meow-star/tests/packages.rs `moving_a_file_changes_the_hash`,
high).

## How I will know it was realised

1. `identical_contents_share_a_cache_entry` in crates/meow-cli/tests/fetch.rs
   passes: two packages with the same contents share one cache directory (from
   crates/meow-cli/tests/fetch.rs:289-316, high).
2. `contents_that_do_not_match_the_lockfile_are_refused` in
   crates/meow-star/tests/packages.rs passes: a cached package edited after
   locking doesn't load (from crates/meow-star/tests/packages.rs:104-138,
   high).

## What this does not settle

- How the cache is pruned: nothing removes an entry that no lockfile pins any
  more (from crates/meow-cli/src/fetch.rs, where `remove_dir_all` touches only
  the staging directory, medium).
- Whether the tree hash is cached against modification times for a large
  package (from https://github.com/retran/meowg1k/pull/157, high).
