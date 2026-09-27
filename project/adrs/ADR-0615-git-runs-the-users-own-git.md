---
id: ADR-0615
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2540]
supersedes: []
---

# 0615. `@std//git` runs the user's own `git` as a subprocess

## Decision

`@std//git` runs the `git` program on the user's path as a subprocess for every
call, and links no Git library (from https://github.com/retran/meowg1k/pull/136
and crates/meow-star/src/capability_git.rs:1-43, high).

Once this is accepted, it works for `diff`, `status`, `log`, `show`, `branch`,
`stage` and `commit`, each passing its words to `git` with no shell, in the
workspace root and under `shell`'s default timeout (from
crates/meow-star/src/capability_git.rs:25-43 and
crates/meow-star/src/capability.rs:296-313, high). Nothing reaches the network:
no `fetch`, `pull`, `push` or `clone`, and nothing moves the repository between
states, such as `checkout` or `reset` (from
https://github.com/retran/meowg1k/pull/136, high).

## Why

The diff a reviewer sees has to be the diff they would see themselves, or the
review is about a different repository, and a library would disagree with the
user's `git` about configuration, hooks, credential helpers and worktrees (from
https://github.com/retran/meowg1k/pull/136, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Link `libgit2` | No process per call (from https://github.com/retran/meowg1k/pull/136, high) | It disagrees with the user's `git` about configuration, hooks, credential helpers and worktrees (from https://github.com/retran/meowg1k/pull/136, high) |
| Do nothing: no `git` module, and a handler runs `git` through `shell` | No module to maintain (reasoned from crates/meow-star/src/capability.rs, low) | The shipped agents need a diff, a status, a log and a commit, and each handler would repeat the argument checks (from https://github.com/retran/meowg1k/pull/136, high) |

## What it costs

A process per call, and a machine without `git` on its path can't use the
module (from https://github.com/retran/meowg1k/pull/136, high). A repository
with no commits fails on `branch` and `log`, because `git` exits non-zero there
(from https://github.com/retran/meowg1k/pull/136, high).

## What would reverse it

- A shipped agent needs so many `git` calls per run that one process each is
  too slow (reasoned from https://github.com/retran/meowg1k/pull/136, low).

## Consequences

- `git.commit` signs exactly when `commit.gpgsign` says to, with no per-call
  choice (from https://github.com/retran/meowg1k/pull/136, high).
- A failure reports `git`'s own message and exit code (from
  crates/meow-star/src/capability_git.rs:33-40, high).

## How I will know it was realised

1. `git_reads_the_repository` and `git_reports_what_the_program_said` in
   crates/meow-star/tests/running.rs pass against a real repository built in
   a temporary directory (from crates/meow-star/tests/running.rs:1084 and
   :1190, high).

## What this does not settle

- How `git` operations that reach the network or change the checked-out state
  would be added, which #136 leaves for their own decision (from
  https://github.com/retran/meowg1k/pull/136, high).
