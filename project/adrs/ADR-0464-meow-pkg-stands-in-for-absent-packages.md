---
id: ADR-0464
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1809, REQ-1811]
supersedes: []
---

# 0464. `meow pkg` stands in for a declared package that isn't there yet

## Decision

`load_for_packages` tolerates a declared package that isn't there yet with a
stand-in module defining exactly the names the `load` statement asked for,
parsed with `AstModule::loads()` (from
https://github.com/meowshed/meowg1k/pull/163, high). Only `meow pkg` reads a
workspace this way, and an undeclared package never stands in (from
https://github.com/meowshed/meowg1k/pull/163, high). The first submission of this
change, https://github.com/meowshed/meowg1k/pull/159, closed without merging, and
this pull request carries the same content (from
https://github.com/meowshed/meowg1k/pull/159, high).

Once this is accepted, `meow pkg update` runs in a workspace whose declared
package is unlocked, writes `meow.lock`, and `meow check` then passes (from
crates/meow-cli/tests/pkg.rs:125, high). The binary tries the strict load first
and falls back to `load_for_packages` only when the subcommand is `pkg` (from
crates/meow-cli/src/lib.rs:79, high). A `@pkg//` file that loads another
unavailable package still fails, because `source_of` reads only `meow.star` and
`//` files (from https://github.com/meowshed/meowg1k/pull/163, high).

## Why

Loading refused because the package wasn't locked, and the only command that
locks it is `meow pkg update`, which couldn't load (from
https://github.com/meowshed/meowg1k/pull/163, high). A stub that defined
everything would let a typo load, and a typo is the other reason a package is
missing (from https://github.com/meowshed/meowg1k/pull/163, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: load strictly for `meow pkg` too | One load path, with no stand-in whose symbols are `None` (reasoned from crates/meow-star/src/loader.rs:383, low) | `meow pkg update` can't run before the package it makes available exists (from https://github.com/meowshed/meowg1k/pull/163, high) |
| A stub that defines every name | It needs no parse of the loading file to learn which names were asked for (reasoned from crates/meow-star/src/loader.rs:172, low) | A typo would load (from https://github.com/meowshed/meowg1k/pull/163, high) |
| Evaluate `meow.star` without its loads to read the declarations | No symbol would ever stand in as `None` (from https://github.com/meowshed/meowg1k/pull/163, high) | The Starlark dialect doesn't offer evaluating a file without its loads (from https://github.com/meowshed/meowg1k/pull/163, high) |

## What it costs

While `meow pkg` runs, a declared-but-absent package's symbols are `None`, so a
`meow.star` that used one during declaration would see `None` rather than fail;
making that impossible needs evaluating a file without its loads, which the
dialect doesn't offer (from https://github.com/meowshed/meowg1k/pull/163, high).

## What would reverse it

A Starlark dialect that can evaluate a file without its loads, or a `meow pkg`
that reads `meow.package` declarations without evaluating `meow.star`, would
remove the need for a stand-in (from
https://github.com/meowshed/meowg1k/pull/163, high).

## Consequences

- Every command other than `meow pkg` still sees the strict error, which names
  `meow pkg update` for an unlocked package, as REQ-1809 asks (from
  crates/meow-cli/src/lib.rs:79 and crates/meow-star/src/package.rs:183,
  high).
- A declared package stands in whatever made the strict load fail, whether it
  is unlocked, uncached or mismatched, so `meow pkg` reads a workspace whose
  cache was edited (from crates/meow-star/src/loader.rs:377, high).
- An undeclared package never stands in, because `meow pkg update` can't fix it
  by fetching (from crates/meow-star/src/loader.rs:137, high).

## How I will know it was realised

`update_then_load_is_the_whole_story` in `crates/meow-cli/tests/pkg.rs` passes:
`meow check` names `meow pkg update` before the lock, `meow pkg update` writes
`meow.lock`, and `meow check` passes after it (from
crates/meow-cli/tests/pkg.rs:125, high). No test checks that a `load` naming a
symbol the package lacks fails once the package is there (reasoned from
crates/meow-star/src/loader.rs:172, low).

## What this does not settle

- A `@pkg//` file that loads another unavailable package can't be stubbed,
  because its source is inside a package that isn't there, and that case fails
  (from https://github.com/meowshed/meowg1k/pull/163, high).
- Whether a `meow.star` that uses a stood-in symbol during declaration should
  be caught while `meow pkg` runs, since today it sees `None` (from
  https://github.com/meowshed/meowg1k/pull/163, high).
