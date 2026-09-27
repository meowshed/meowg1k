---
id: ADR-0104
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2400, REQ-2401, REQ-2402]
supersedes: []
---

# 0104. A workspace's configuration is read alone and never merged with a global one

## Decision

Discovery walks up from the working directory to the first `.meow/meow.star` and
stops, and the project's configuration wins outright with no merging against a
global one (from docs/design/0.3.0-starlark-api.md section 2, high). A project
configuration means the global one isn't read at all (from docs/philosophy.md
section 6, high). `Workspace::discover` implements this and reads no file
outside the workspace it finds (from crates/meow-star/src/workspace.rs:21-47,
high).

## Why

A global setting bleeding into a project that didn't restate it makes the
workspace behave differently on two machines (from docs/philosophy.md section 6,
high). Merging would mean a run behaves differently depending on a file outside
the repository, and no amount of documentation makes that debuggable (from
crates/meow-star/src/workspace.rs:24-27, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: keep v0.2.1's global `~/.config/meowg1k/init.star` that a project overrides | Providers, models and presets are defined once and are available in every project (from `git show v0.2.1:cmd/init.go`, lines 32-34, high) | A global setting that bleeds into a project makes the workspace behave differently on two machines (from docs/philosophy.md section 6, high) |
| Keep searching past the first match and merge every `.meow/` up the tree | A monorepo could share one parent configuration across nested projects (reasoned from crates/meow-star/tests/loading.rs:47-62, low) | A parent's setting would bleed into a nested project that didn't restate it, which is the failure the principle names for a global file (reasoned from docs/philosophy.md section 6, low) |

A third layered design isn't recorded in the design documents, the code or the
pull request history.

## What it costs

Every workspace restates its providers and models, because nothing is inherited
from the machine (reasoned from docs/design/0.3.0-starlark-api.md section 2,
low). Credentials are the one thing kept outside the workspace, in
`~/.meow/auth.json`, so a project doesn't have to name keys to work (from
docs/design/0.3.0-starlark-api.md section 4.1, high).

## What would reverse it

- Users repeatedly copy the same provider and model declarations between
  workspaces and a declared package can't carry them, which is the need the
  v0.2.1 global file served (reasoned from docs/spec/packages.md Scope and
  `git show v0.2.1:cmd/init.go`, low).

## Consequences

Credentials are the exception the design keeps outside the workspace: they live
in `~/.meow/auth.json`, which a script never writes (from
docs/design/0.3.0-starlark-api.md section 4.1, high). Starlark shared between
workspaces arrives as a declared, pinned package loaded through `@<pkg>//`, not
as a global layer (from docs/spec/packages.md Scope and [R-PKG-001], high). A
run outside any workspace fails naming every directory searched and suggests
`meow init` (from crates/meow-star/tests/loading.rs:64-80, high).

## How I will know it was realised

1. `discovery_takes_the_nearest_ancestor_and_merges_nothing` in
   crates/meow-star/tests/loading.rs passes: with a workspace nested inside
   another, discovery from a deep directory returns the inner one (from
   crates/meow-star/tests/loading.rs:47-62, high).
2. No code path in `meow-star` or `meow-cli` reads a configuration file from
   the home directory other than `auth.json` and `trust.json` (reasoned from
   docs/design/0.3.0-starlark-api.md section 4.1, medium).

## What this does not settle

- Where credentials and trust live, which is outside the workspace on purpose
  and belongs to the credential decisions (from
  docs/design/0.3.0-starlark-api.md section 4.1, high).
- How `--workspace <path>` interacts with discovery: the flag makes the binary
  act as if run from that path (from docs/design/0.3.0-tui.md section 2.3,
  high).
