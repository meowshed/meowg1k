---
id: BUG-0030
artifact: bug
status: approved
severity: minor
violates: REQ-2442
found: 2026-09-27
revised: 2026-09-27
issue:
---

# `search.text` walks with defaults when the declared index can't be built

When a workspace declares an index but its model or its provider is missing, the
CLI falls back to a search port that walks with `Walk::default()`, not with the
walk the index declares. `search.text` and `search.files` then report files the
index would skip, such as files over the declared `max_bytes`.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare `meow.index(model = "embed", max_bytes = 1000)` where the provider of
   model `embed` isn't declared, or where `embed` isn't a declared model.
2. Put a 5000-byte text file in the workspace.
3. Run a command whose handler calls `search.files("*")`.

The 5000-byte file is listed. The index, built with the declared walk, would
skip it. This was traced through the code and not run.

## What the system does

`searcher` in `crates/meow-cli/src/wire.rs:1249-1289` returns
`crate::index::Unindexed::new(workspace.root(), why)` on every failure path,
including after `configure` has already produced the declared walk.
`Unindexed::new` at `crates/meow-cli/src/index.rs:392-399` builds its walker
from `Walk::default()`.

## What it should do, and why

REQ-2442: "`search.text` and `search.files` MUST obey the same walk the index
obeys, so a file the index ignores is a file they do not report."

## Triage

The defect enters in `meow-cli`'s `searcher`, which drops the walk it computed
when it falls back. It is minor: it needs a declared index whose model or
provider is missing, and the only walk setting today is `max_bytes`. The fix is
to pass the declared walk to `Unindexed` whenever the index is declared.

## Closed by

A test in `crates/meow-cli/tests/search.rs`, named for example
`search_files_obeys_the_declared_walk_when_the_index_cannot_open`, that runs the
steps above and expects the large file left out.
