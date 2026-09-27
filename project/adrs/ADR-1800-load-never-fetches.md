---
id: ADR-1800
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1809, REQ-1810]
supersedes: []
---

# 1800. A load never fetches

## Decision

A load consults the lockfile and never reaches the network, and fetching is a
command a person runs (from docs/spec/packages.md [R-PKG-012], high).

Once this is accepted, two runs of the same commit load the same bytes, and a
workspace that has fetched once loads offline (from
crates/meow-cli/tests/pkg.rs `a_fetched_workspace_loads_offline`, high). A
fresh checkout still doesn't run until a person runs `meow pkg fetch`, which
the load error names (from crates/meow-star/src/package.rs:189-196, high).

## Why

Resolving on demand makes a build depend on the network and on when it ran (from
docs/spec/packages.md, Decisions, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Resolve and fetch on demand when a load names a package | It is what every package manager did first (from docs/spec/packages.md, Decisions, high), and a fresh checkout runs with no extra command (reasoned from crates/meow-star/src/package.rs:189-196, low). | It makes a build depend on the network and on when it ran, and package managers stopped doing it (from docs/spec/packages.md, Decisions, high). |
| Do nothing: no `@<pkg>//` scheme, as in v0.2.x | No code from somebody else reaches the loader and no network code sits behind `load` (reasoned from docs/spec/packages.md, Changes from v0.2.x, low). | The design promises a third `load` scheme for Starlark a workspace didn't write (from docs/design/0.3.0-starlark-api.md section 10, high). |

Neither the source nor the history names a third option for how a load
resolves a package (from docs/spec/packages.md and
https://github.com/meowshed/meowg1k/pull/157, medium).

## What it costs

A person runs `meow pkg update` or `meow pkg fetch` before the first load, and
a load that finds a package unlocked or uncached fails naming that command
(from crates/meow-star/src/package.rs:180-196, high). `meow pkg update` has to
read the declarations while their loads refuse, so the loader stands in a
module of `None` names for a declared, absent package while `meow pkg` runs
(from crates/meow-star/src/loader.rs:67-79 and
https://github.com/meowshed/meowg1k/pull/163, high).

## What would reverse it

- A fetch at load time that could be shown to reach only the pinned hash and
  verify it before evaluating anything, so that a run stayed a function of the
  commit, would remove the reason REQ-1810 gives (reasoned from
  docs/spec/packages.md [R-PKG-012] and crates/meow-star/src/package.rs:199-210,
  low).

## Consequences

Fetching is a command a person runs, and the lockfile is what a load consults
(from docs/spec/packages.md, Decisions, high). A load is testable with no
network at all, so every loader test puts the package in the cache by hand
(from crates/meow-star/tests/packages.rs:4-8, high). The network half lives in
`meow-cli` and the loading half in `meow-star` (from
https://github.com/meowshed/meowg1k/pull/162, high).

## How I will know it was realised

1. `a_package_that_is_not_locked_says_to_lock_it` and
   `a_package_that_is_not_cached_says_to_fetch_it` in
   crates/meow-star/tests/packages.rs pass: the load fails naming the command
   and makes no request (from crates/meow-star/tests/packages.rs:140-174,
   high).
2. `a_fetched_workspace_loads_offline` in crates/meow-cli/tests/pkg.rs passes
   with the server dropped before the load (from crates/meow-cli/tests/pkg.rs
   and https://github.com/meowshed/meowg1k/pull/162, high).

## What this does not settle

- Which sources a fetch accepts: today a gzipped tarball over HTTP and nothing
  else, with no git and no local path (from
  https://github.com/meowshed/meowg1k/pull/162, high).
- Version ranges: `meow pkg update` fetches exactly the declared version (from
  https://github.com/meowshed/meowg1k/pull/162, high).
- The fetch's 64 MiB cap and 120 second deadline, which were chosen and aren't
  specified (from https://github.com/meowshed/meowg1k/pull/162, high).
