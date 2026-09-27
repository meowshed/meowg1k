---
id: BUG-0014
artifact: bug
status: approved
severity: minor
violates: REQ-1821
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A package loads its own sibling by package name, not by a relative path

REQ-1821 says a package file loads another file of the same package "by a
relative path". The loader has no relative form: a package loads its sibling as
`@acme//inner.star`, which is what the test citing REQ-1821 does, and a bare
`inner.star` or `./inner.star` is refused.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Read `a_package_reaches_the_runtime_modules_and_its_own_files` in
   `crates/meow-star/tests/packages.rs:239-280`: `models.star` loads
   `@acme//inner.star`.
2. Change that load to `load("inner.star", "helper")` and run
   `cargo test -p meow-star --test packages`.

The test fails with "`inner.star` is not a load path".

## What the system does

`crates/meow-star/src/loader.rs:90-118` accepts only `@<pkg>//<path>` and
`//<path>`, and refuses anything else as "not a load path".

## What it should do, and why

REQ-1821: "A file in a package MUST be able to `load("@std//...")` and to load
another file in the same package by a relative path."

## Triage

The defect enters where REQ-1821 and the loader disagree on the form of a
sibling load. The requirement is probably the part that is wrong: `@acme//` works,
is what the test uses, and needs no second path syntax. Amending REQ-1821 to
name `@<pkg>//<path>` would close this. A package that names itself does depend
on the name the workspace gave it in `meow.package`, which is the argument for a
relative form, and the amendment should weigh it. It is minor because a package
author can load a sibling today.

## Closed by

`a_package_reaches_the_runtime_modules_and_its_own_files`, citing the amended
REQ-1821, or a new test that loads a sibling by the relative form if the code
changes instead.
