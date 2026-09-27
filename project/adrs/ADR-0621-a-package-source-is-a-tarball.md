---
id: ADR-0621
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1825]
supersedes: []
---

# 0621. A package source is a gzipped tarball over HTTP

## Decision

A package's source is a URL that answers with a gzipped tar archive, with
`{version}` replaced by the declared version, and nothing else: no git and no
local paths (from https://github.com/retran/meowg1k/pull/162 and
crates/meow-cli/src/fetch.rs:70-77 and :184-192, high).

Once this is accepted, `meow pkg update` and `meow pkg fetch` work for such a
source, and `update` fetches exactly the declared version, since no range
syntax is specified (from https://github.com/retran/meowg1k/pull/162, high).

## Why

Git would need either a large dependency or shelling out, and that choice
deserves its own thought (from https://github.com/retran/meowg1k/pull/162,
high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| A git repository as a source | Most Starlark libraries live in a repository, with no archive to publish (reasoned from https://github.com/retran/meowg1k/pull/162, low) | It needs a large dependency or a subprocess, a choice left for its own decision (from https://github.com/retran/meowg1k/pull/162, high) |
| A local path as a source | A package under development loads without publishing (reasoned from https://github.com/retran/meowg1k/pull/162, low) | Not taken in #162, which names it among what isn't covered (from https://github.com/retran/meowg1k/pull/162, high) |

Doing nothing isn't an option, because fetching was new in #162 and needed one
source kind (from https://github.com/retran/meowg1k/pull/162, high).

## What it costs

A package author publishes each release as a tarball at a stable URL (reasoned
from crates/meow-cli/src/fetch.rs:70-77, low).

## What would reverse it

- Packages in use are published only as repositories (reasoned from
  https://github.com/retran/meowg1k/pull/162, low).

## Consequences

- A source that answers with anything but a gzipped tar fails when it is
  unpacked (reasoned from crates/meow-cli/src/fetch.rs:190-200, low).

## How I will know it was realised

1. `update_writes_a_pin_for_what_arrived` and
   `a_version_is_put_into_the_source` in crates/meow-cli/tests/fetch.rs pass
   (from crates/meow-cli/tests/fetch.rs:131-155 and :271-287, high).

## What this does not settle

- Version ranges in `update` (from https://github.com/retran/meowg1k/pull/162,
  high).
