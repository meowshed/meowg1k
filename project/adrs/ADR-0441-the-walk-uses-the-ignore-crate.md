---
id: ADR-0441
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1400]
supersedes: []
---

# 0441. The workspace walk uses the `ignore` crate

## Decision

The walk uses `ignore`, ripgrep's walker (from
https://github.com/retran/meowg1k/pull/130, high).

Once this is accepted, the index walk honours `.gitignore` and
`.git/info/exclude` whether or not the workspace is a repository, and reads
`.meowignore` after them so a negation in it wins (from
crates/meow-index/src/walk.rs:110, high). It doesn't honour the user's global
git excludes or a `.gitignore` in a directory above the workspace, because the
walk sets `git_global(false)` and `parents(false)` (from
crates/meow-index/src/walk.rs:113, high).

## Why

That is where the `.gitignore` semantics anybody expects actually live (from
https://github.com/retran/meowg1k/pull/130, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: reimplement the `.gitignore` semantics, as v0.2.x did with regular expressions in `internal/adapters/gitignore` | No new dependency, and every rule is code the project can read (from `git show v0.2.1:internal/adapters/gitignore/gitignore.go`, medium) | It reimplements the part users notice when it is subtly wrong (from https://github.com/retran/meowg1k/pull/130, high) |
| Ask git for the file list with `git ls-files` | Git's own answer, exact by definition (reasoned from crates/meow-index/src/walk.rs:117, low) | A workspace need not be a repository, and its `.gitignore` still says what doesn't belong in the index (from crates/meow-index/src/walk.rs:117, high) |

The code and the history name no third option.

## What it costs

One new dependency, `ignore` 0.4, which brings `globset`, `regex-automata`,
`walkdir`, `crossbeam-deque` and four more crates into the build (from
https://github.com/retran/meowg1k/pull/130 and Cargo.lock, high). A negation in
`.meowignore` for a file inside a directory `.gitignore` excludes still needs a
second walk, because the crate follows git's rule that such a file can't be
re-included (from https://github.com/retran/meowg1k/pull/130, high).

## What would reverse it

The `ignore` crate's matching is observed to differ from git's on a pattern a
user relies on, or the crate stops being maintained and `cargo deny` reports an
advisory against it with no safe upgrade (reasoned from deny.toml, low).

## Consequences

- The walk's rules are the crate's, so `.gitignore` behaves as ripgrep reads it
  (from https://github.com/retran/meowg1k/pull/130, high).
- `.meowignore` is a custom ignore file name added after `.gitignore`, so its
  lines take precedence (from crates/meow-index/src/walk.rs:122, high).
- The first walk leaves the crate's `ignore(true)` default in place, so a
  `.ignore` file in the workspace also excludes files, which no requirement
  mentions (reasoned from crates/meow-index/src/walk.rs:110 against the
  crate's `WalkBuilder` defaults, medium).
- The second walk, over the paths `.meowignore` re-includes, turns every ignore
  source off (from crates/meow-index/src/walk.rs:201, high).

## How I will know it was realised

`a_gitignored_file_is_not_indexed` and
`meowignore_can_re_include_what_git_excludes` in
`crates/meow-index/tests/walk.rs` pass against a real directory (from
crates/meow-index/tests/walk.rs:46, high).

## What this does not settle

- Whether a global git excludes file or a `.gitignore` above the workspace
  should apply to the index (from crates/meow-index/src/walk.rs:113, high).
- How a `.meowignore` negation inside an excluded directory is found, which
  ADR-0117 settles with a second walk (from
  https://github.com/retran/meowg1k/pull/130, high).
