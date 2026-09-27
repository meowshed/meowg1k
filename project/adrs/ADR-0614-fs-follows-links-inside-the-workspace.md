---
id: ADR-0614
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2539]
supersedes: []
---

# 0614. `fs` checks a relative path lexically and follows links inside it

## Decision

`fs` checks a relative path lexically, refusing `..`, and doesn't resolve
symbolic links, so a link inside the workspace that points outside it is
followed; an absolute path is canonicalised and refused when it lands outside
(from https://github.com/meowshed/meowg1k/pull/135 and
crates/meow-star/src/capability.rs:38-61, high).

Once this is accepted, it works as stated for every `fs` call that takes a
path (from crates/meow-star/src/capability.rs:73-205, high).

## Why

Catching a link that points out means resolving every path before every call,
a system call per call, and the case it protects against is a link somebody put
in their own repository (from https://github.com/meowshed/meowg1k/pull/135,
high). Checking before touching the disk also reports a `..` that happens to
land back inside, and keeps a link from deciding where the boundary is (from
crates/meow-star/src/capability.rs:38-42, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Resolve every path and refuse one whose target is outside | A link inside the workspace can't reach outside it (from https://github.com/meowshed/meowg1k/pull/135, high) | A system call per call, against a link the repository's own author placed (from https://github.com/meowshed/meowg1k/pull/135, high) |
| Do nothing: no confinement at all | No check on any call (reasoned from crates/meow-star/src/capability.rs:44, low) | A path that escapes the workspace is a mistake whoever made it, and before the call is the cheapest place to notice (from https://github.com/meowshed/meowg1k/pull/135, high) |

## What it costs

A handler can read or write outside the workspace through a link inside it
(from https://github.com/meowshed/meowg1k/pull/135, high). Indexing refuses such
a link by REQ-1407, so `fs` and the index disagree about the same file
(reasoned from docs/requirements/REQ-1407-no-symlink-leaving-workspace.md,
low).

## What would reverse it

- `fs` is exposed to paths a model chose, without the policy resolving them
  first (reasoned from crates/meow-star/src/run.rs:571-582, low).

## Consequences

- A tool call from a model is still judged on its resolved path by the policy
  before the handler runs (from crates/meow-star/src/run.rs:571-582 and
  ADR-0401, high).

## How I will know it was realised

1. `fs_refuses_a_path_outside_the_workspace` in
   crates/meow-star/tests/running.rs passes, refusing `..` lexically and an
   absolute path outside (from crates/meow-star/tests/running.rs:846-853,
   high). No test creates a link; one that reads through a link inside the
   workspace pointing out, given a relative path, would show REQ-2539.

## What this does not settle

- Whether `fs.glob`, which follows linked directories while it walks, should
  stop at a link that leaves the workspace (reasoned from
  crates/meow-star/src/capability.rs:166, low).
