---
id: SPC-1800
artifact: spec
status: live
revised: 2026-09-27
checked-at:
states: [REQ-1800, REQ-1801, REQ-1802, REQ-1803, REQ-1804, REQ-1805, REQ-1806, REQ-1807, REQ-1808, REQ-1809, REQ-1810, REQ-1811, REQ-1812, REQ-1813, REQ-1814, REQ-1815, REQ-1816, REQ-1817, REQ-1818, REQ-1819, REQ-1820, REQ-1821, REQ-1822, REQ-1823, REQ-1824, REQ-1825, REQ-1826, REQ-1827, REQ-1828, REQ-1829]
---

# Packages: Starlark a workspace did not write

## Scope

This covers fetching Starlark a workspace did not write, pinning it so that the
same workspace loads the same bytes tomorrow, verifying it before it runs, and
keeping it where a second checkout needn't fetch it again (from
docs/spec/packages.md, high).

A package is the third `load` scheme. `@std//` is Rust in the binary and `//` is
a file the workspace wrote; `@<pkg>//` is neither, because it arrives over a
network from somebody else (from docs/spec/packages.md, high). The other two
schemes belong to SPC-2400.

## Boundary

- A declaration in `.meow/meow.star` names each package (from
  docs/spec/packages.md, high).
- A lockfile at `.meow/meow.lock` is committed (from docs/spec/packages.md,
  high).
- A cache under `.meow/.data/pkg/` isn't committed (from docs/spec/packages.md,
  high).
- `meow pkg update` and `meow pkg fetch` are the two commands that reach the
  network (from docs/spec/packages.md [R-PKG-013], high).

Everything else is the loader resolving a scheme (from docs/spec/packages.md,
high).

## Behaviour

### Declaring

- A workspace declares every package it loads, with a name, a source and a
  version [REQ-1800] (from docs/spec/packages.md [R-PKG-001], high).

### Pinning

- `.meow/meow.lock` pins every package by its resolved version and the SHA-256
  of its contents [REQ-1805] (from docs/spec/packages.md [R-PKG-010], high).
- The same declarations and the same upstream produce the same lockfile bytes,
  so a diff shows a dependency change and nothing else [REQ-1806] (from
  docs/spec/packages.md [R-PKG-010], high).
- Loading verifies the hash of what it is about to run against the lockfile
  [REQ-1807] (from docs/spec/packages.md [R-PKG-011], high).
- `meow pkg update` re-resolves versions and rewrites the lockfile [REQ-1811]
  (from docs/spec/packages.md [R-PKG-013], high).
- `meow pkg fetch` downloads what the lockfile already pins and leaves the
  lockfile unchanged [REQ-1812] [REQ-1813] (from docs/spec/packages.md
  [R-PKG-013], high).

### Fetching and caching

- A package whose contents are in the cache and match the lockfile loads with no
  network access, so a workspace that has fetched once works offline [REQ-1814]
  [REQ-1815] (from docs/spec/packages.md [R-PKG-020], high).
- The cache is keyed by hash, so two workspaces pinning the same package share
  one copy and a changed pin is never served the old bytes [REQ-1816] (from
  docs/spec/packages.md [R-PKG-021], high).
- A fetch is atomic, carries a deadline and can be interrupted [REQ-1817]
  [REQ-1819] [REQ-1820] (from docs/spec/packages.md [R-PKG-022] [R-PKG-023],
  high).
- A package source is a gzipped tar archive fetched over HTTP or HTTPS
  [REQ-1825], and a download that hasn't finished in 120 seconds fails
  [REQ-1829] (from https://github.com/retran/meowg1k/pull/162, high).

### What a package may do

- A file in a package loads `@std//...` and loads another file in the same
  package by a relative path [REQ-1821] (from docs/spec/packages.md [R-PKG-030],
  high).
- Code from a package runs under the same rules as code the workspace wrote: the
  declaration phase reaches nothing, and what a model decides still passes
  through policy [REQ-1824] (from docs/spec/packages.md [R-PKG-033], high).

## Failure paths

- A `load("@<pkg>//<path>", ...)` naming an undeclared package fails saying so
  and fetches nothing [REQ-1801] [REQ-1802] (from docs/spec/packages.md
  [R-PKG-001], high).
- Two packages claiming the same name are refused at load time, and the error
  names both [REQ-1803] (from docs/spec/packages.md [R-PKG-002], high).
- A package named `std` is refused, so a fetched package can't shadow the
  runtime modules [REQ-1804] (from docs/spec/packages.md [R-PKG-003], high).
- A hash that differs from the lockfile fails the load before anything is
  evaluated, naming the package, the expected hash and the one found [REQ-1808]
  (from docs/spec/packages.md [R-PKG-011], high).
- A declared package absent from the lockfile fails the load, naming the command
  that writes one, and the load doesn't fetch [REQ-1809] [REQ-1810] (from
  docs/spec/packages.md [R-PKG-012], high).
- An archive holding an entry whose path contains `..` or starts at a root is
  refused [REQ-1826], and so is one holding an entry the unpacker declines to
  write, so an escaping archive never pins an empty package [REQ-1827] (from
  https://github.com/retran/meowg1k/pull/162, high).
- An archive larger than 64 MiB is refused [REQ-1828] (from
  https://github.com/retran/meowg1k/pull/162, high).
- An interrupted download leaves nothing the next run takes for a complete
  package [REQ-1818] (from docs/spec/packages.md [R-PKG-022], high).
- A file in a package that loads `//<path>`, the host workspace's own tree,
  fails [REQ-1822] (from docs/spec/packages.md [R-PKG-031], high).
- A package that loads another package the host workspace didn't declare fails
  [REQ-1823] (from docs/spec/packages.md [R-PKG-032], high).
