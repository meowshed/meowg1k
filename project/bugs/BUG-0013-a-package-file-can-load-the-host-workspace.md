---
id: BUG-0013
artifact: bug
status: approved
severity: major
violates: REQ-1822
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A package file can load the host workspace's files with `//`

`Loader::resolve` doesn't know which file is loading. Every `//<path>` load goes
to the workspace's `.meow/`, including one written inside a package, so a
package can read and depend on the tree of whichever workspace loaded it.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments. In a test beside
`a_package_path_cannot_climb_out` in `crates/meow-star/tests/packages.rs`:

1. Use `with_package` to create a workspace whose `.meow/secret.star` holds
   `x = 1` and whose `meow.star` declares package `acme` and loads
   `@acme//models.star`.
2. Give the package a `models.star` holding `load("//secret.star", "x")`.
3. Lock the package and call `load_at`.

The load succeeds. It should fail with an error saying a package can't load the
host workspace.

## What the system does

`crates/meow-star/src/loader.rs:90-118` routes any path starting with `//` to
`self.local`, with no record of whether the caller is a package file. The doc
comment at `crates/meow-star/src/loader.rs:120-126` says a package's own `//`
loads "resolve against the workspace", which is the behaviour REQ-1822 forbids.

## What it should do, and why

REQ-1822: "A file in a package MUST NOT be able to load `//<path>`, which is the
host workspace's own tree. A dependency reaching into the workspace that
depends on it inverts the direction and makes the package's behaviour depend on
who loaded it."

## Triage

The defect enters in `meow-star`'s loader, which resolves a load path without
the loading module's origin. It is major because the requirement's main
behaviour doesn't happen. The fix is to pass the loading module's key into
`resolve` and refuse `//` when it starts with `@`.

## Closed by

A test in `crates/meow-star/tests/packages.rs`, named for example
`a_package_cannot_load_the_host_workspace`, that runs the steps above and
expects the refusal.
