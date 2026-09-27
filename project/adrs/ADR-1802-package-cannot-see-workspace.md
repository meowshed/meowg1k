---
id: ADR-1802
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1822]
supersedes: []
---

# 1802. A package cannot see the workspace that loads it

## Decision

A file in a package can't load `//<path>`, the host workspace's own tree (from
docs/spec/packages.md [R-PKG-031], high).

Once this is accepted, a package behaves the same whichever workspace loads it,
and anything it needs from its caller arrives as an argument at run time
(reasoned from docs/spec/packages.md, Decisions, low). The loader doesn't
enforce it yet: a `//` load resolves against the workspace whatever file makes
it, so today a package file can load `//<path>` (from
crates/meow-star/src/loader.rs:110-112 and 121-126, high).

## Why

A package that loads from whoever uses it is untestable on its own, and its
behaviour becomes a function of its caller (from docs/spec/packages.md,
Decisions, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Let a package load files from the workspace that uses it, such as `//lib/style.md` | It reads well in a small example (from docs/spec/packages.md, Decisions, high). | It makes the package untestable on its own and its behaviour a function of its caller (from docs/spec/packages.md, Decisions, high). |
| Do nothing: resolve every `//` load against the workspace, as the loader does today | No check of which file makes the load, so the loader stays one dispatch on the prefix (from crates/meow-star/src/loader.rs:90-118, high). | It is the first option in effect, and loses for the same reason (reasoned from crates/meow-star/src/loader.rs:121-126, medium). |

Neither the source nor the history names a third option (from
docs/spec/packages.md and https://github.com/retran/meowg1k/pull/157, medium).

## What it costs

A package can't reuse a workspace's shared prompt fragments or style files by
path, and a workspace that wants a package to use one passes it in (reasoned
from docs/spec/packages.md, Decisions, low). The loader has to know which
scheme the file making a `//` load came from, which `FileLoader::load` doesn't
receive today (from crates/meow-star/src/loader.rs:82-86, high).

## What would reverse it

- A package that a real workspace needs and that can't be written without
  reading its caller's files, where passing the content as an argument doesn't
  serve, would reopen the question (reasoned from docs/spec/packages.md,
  Decisions, low).

## Consequences

A package is testable on its own, with nothing from its caller (from
docs/spec/packages.md, Decisions, high). A package names its own files through
its own scheme, `@<pkg>//<path>`, and the tests do so (from
crates/meow-star/tests/packages.rs:239-280, high).

## How I will know it was realised

1. A test in crates/meow-star/tests/packages.rs loads `//<path>` from a
   package file and fails, while the same file loads through `//` from
   `meow.star`. No such test exists yet: `a_package_path_cannot_climb_out`
   covers `..` inside a package path and not a `//` load (from
   crates/meow-star/tests/packages.rs:282-308, high).

## What this does not settle

- How the loader tells a package file's `//` load from a workspace file's,
  since `FileLoader::load` receives only the path (from
  crates/meow-star/src/loader.rs:82-86, high).
- Whether a package loads its own files by a relative path, as REQ-1821 says,
  or by `@<pkg>//<path>`, as the code and the tests do (from
  docs/requirements/REQ-1821-package-loads-std-and-siblings.md and
  crates/meow-star/tests/packages.rs:239-280, high).
