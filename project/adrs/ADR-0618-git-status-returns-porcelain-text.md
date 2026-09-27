---
id: ADR-0618
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2543]
supersedes: []
---

# 0618. `git.status` returns `git`'s porcelain format as text

## Decision

`git.status()` runs `git status --porcelain=v1` and returns what it prints, one
path per line, as a string (from https://github.com/meowshed/meowg1k/pull/136 and
crates/meow-star/src/capability_git.rs:88-98, high). It doesn't parse the lines
into values, so REQ-2543 states the text and not a structure (from
crates/meow-star/src/capability_git.rs:91-98, high).

Once this is accepted, it works as stated (from
crates/meow-star/tests/running.rs:1084-1118, high).

## Why

The porcelain format is the one `git` promises not to change between versions;
the human format is prettier and isn't a contract (from
https://github.com/meowshed/meowg1k/pull/136, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| The human format of `git status` | Prettier to read (from https://github.com/meowshed/meowg1k/pull/136, high) | It isn't a contract, so a handler that reads it can break on a `git` upgrade (from https://github.com/meowshed/meowg1k/pull/136, high) |

Doing nothing isn't an option, because `git.status` was new in #136 and had to
return one format or another (from https://github.com/meowshed/meowg1k/pull/136,
high).

## What it costs

A handler that wants the state of one file splits the lines and reads the
two-letter code itself (reasoned from
crates/meow-star/src/capability_git.rs:88-98, low).

## What would reverse it

- Handlers in `.meow/` or a shipped agent repeat the same parsing of the lines
  (reasoned from .meow/meow.star, low).

## Consequences

- A model given `git.status` as a tool reads the porcelain lines directly
  (reasoned from crates/meow-star/src/capability_git.rs:91, low).

## How I will know it was realised

1. `git_reads_the_repository` in crates/meow-star/tests/running.rs passes; it
   expects `?? new.txt` for an untracked file (from
   crates/meow-star/tests/running.rs:1084-1118, high).

## What this does not settle

- Whether a parsed form is added beside the text later (reasoned from
  crates/meow-star/src/capability_git.rs:88-98, low).
