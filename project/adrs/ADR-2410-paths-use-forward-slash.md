---
id: ADR-2410
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2472, REQ-2473, REQ-2474]
supersedes: []
---

# 2410. Paths are written with `/` on every platform

## Decision

`path` speaks `/` on every platform, accepts `\` in its input and never returns
it (from docs/spec/starlark.md [R-STAR-029], high).

## Why

`search.files` and `fs.glob` report `/`, so a handler that built a path with the
native separator and compared it with a reported one matched on Unix and failed
on Windows, with nothing saying why. One separator on the surface removes that
class of defect (from docs/spec/starlark.md, Decisions, high). The Windows CI
runner found the defect in `path.join("a", "b")`, which came back as `a\b` (from
https://github.com/retran/meowg1k/pull/149, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Do nothing: use the native separator, as v0.2.x's `path` did through `filepath` | It is the obvious choice (from docs/spec/starlark.md, Decisions, high), and it is what the operating system and the programs `shell` runs print (reasoned from v0.2.1:internal/core/starlark/module_path.go:50-144, low). | A path built with it fails to compare with one `search.files` or `fs.glob` reported, on Windows only and silently (from docs/spec/starlark.md, Decisions, high). |
| Rewrite every `\` to `/` on every platform | REQ-2474 would hold on Unix too, because no `path` result could ever contain `\` (reasoned from crates/meow-star/src/modules.rs:757-776, low). | On Unix a backslash is an ordinary character in a file name, so a file really called `a\b` would be renamed (from https://github.com/retran/meowg1k/pull/149, high). |

Neither the code, the history nor the design documents name a further option.

## What it costs

- On Unix, `path` returns `\` when a name contains one: `base("a\\b\\c.rs")`
  gives `a\b\c.rs` (from crates/meow-star/tests/running.rs,
  `a_path_given_with_a_backslash_is_understood`, high). REQ-2474 therefore holds
  only where `\` is a separator.
- `path.rel` now returns a path that isn't under the base with its separators
  rewritten, where it used to return it unchanged (from
  https://github.com/retran/meowg1k/pull/149, high).
- Windows accepts `/` in a path, so emitting it gives nothing else up (from
  docs/spec/starlark.md, Decisions, high).

## What would reverse it

A supported platform that doesn't accept `/` as a separator in a path given to
the operating system would make the chosen separator wrong there (reasoned
from docs/spec/starlark.md, Decisions, low).

## Consequences

- `path` builds with the native separator internally and rewrites to `/` on
  the way out, and rewrites `/` to the native separator on the way in (from
  crates/meow-star/src/modules.rs:757-776, high).
- A path that arrives from `shell.capture` is whatever the program printed; a
  handler compares it after passing it through `path.join` or `path.base`,
  which now normalise it (from https://github.com/retran/meowg1k/pull/149,
  high).

## How I will know it was realised

The tests `paths_are_written_with_one_separator` and
`a_path_given_with_a_backslash_is_understood` in
crates/meow-star/tests/running.rs pass on the `windows-latest` runner (from
.github/workflows/ci.yaml:69, high). On Unix the first passes whether or not
the fix is there, so only the Windows run proves it (from
https://github.com/retran/meowg1k/pull/149, high).

## What this does not settle

- `fs` and `shell` take paths and hand them to the operating system unchanged;
  they aren't the surface a handler compares strings on (from
  https://github.com/retran/meowg1k/pull/149, high).
- Whether REQ-2474 should say "where `\` is a separator", so the Unix case in
  What it costs stops contradicting it (reasoned from
  crates/meow-star/tests/running.rs, low).
