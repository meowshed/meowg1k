---
id: ADR-0617
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2542]
supersedes: []
---

# 0617. `git.commit` takes only what is staged

## Decision

`git.commit(message)` runs `git commit --message <message> --` and so commits
the index and nothing in the working tree, then returns the new commit's short
hash (from https://github.com/retran/meowg1k/pull/136 and
crates/meow-star/src/capability_git.rs:164-200, high). An empty message is
refused (from crates/meow-star/src/capability_git.rs:172-174, high).

Once this is accepted, it works as stated (from
crates/meow-star/tests/running.rs:1124, high).

## Why

A commit that also picked up whatever happened to be in the working tree is a
commit nobody reviewed (from https://github.com/retran/meowg1k/pull/136, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Commit the working tree too, as `git commit -a` does | One call where a handler would otherwise stage first (reasoned from crates/meow-star/src/capability_git.rs:150-160, low) | It commits changes nobody staged and so nobody reviewed (from https://github.com/retran/meowg1k/pull/136, high) |
| Take a list of paths to commit | A handler names exactly what goes in (reasoned from crates/meow-star/src/capability_git.rs:150, low) | `git.stage` already names the paths, and a second way to choose them would bypass the review of what is staged (reasoned from .meow/meow.star, low) |

Doing nothing isn't an option, because `git.commit` was new in #136 and had to
take something (from https://github.com/retran/meowg1k/pull/136, high).

## What it costs

A handler that wants to commit a change stages it first, with `git.stage`
(from crates/meow-star/src/capability_git.rs:150-160, high).

## What would reverse it

- A shipped agent has to commit changes it can't stage one by one (reasoned
  from https://github.com/retran/meowg1k/pull/136, low).

## Consequences

- `meow commit` in this repository commits what the person staged, which is
  what it writes a message for (from .meow/meow.star:180-181, high).

## How I will know it was realised

1. `git_commits_only_what_is_staged` in crates/meow-star/tests/running.rs
   passes; it checks that a loose file is still loose afterwards (from
   crates/meow-star/tests/running.rs:1124 and
   https://github.com/retran/meowg1k/pull/136, high).

## What this does not settle

- Signing per call: the commit signs when `commit.gpgsign` says so (from
  https://github.com/retran/meowg1k/pull/136, high).
