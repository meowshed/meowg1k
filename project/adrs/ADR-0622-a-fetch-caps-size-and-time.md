---
id: ADR-0622
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1828, REQ-1829]
supersedes: []
---

# 0622. A fetch refuses an archive over 64 MiB and gives up after 120 seconds

## Decision

A fetch refuses an archive larger than 64 MiB and gives up when the download
hasn't finished in 120 seconds (from https://github.com/retran/meowg1k/pull/162
and crates/meow-cli/src/fetch.rs:17-25 and :97-100, high). Both numbers are
chosen, not specified: REQ-1819 asks for a deadline and says nothing about how
long (from https://github.com/retran/meowg1k/pull/162, high).

Once this is accepted, both limits work, but the size is checked after the
whole body has been read into memory, so the cap protects the disk and not the
memory (from crates/meow-cli/src/fetch.rs:122-138, high).

## Why

A package is Starlark, and something claiming to be one and arriving as a
gigabyte isn't a package; refusing it is cheaper than filling a disk to find
out (from crates/meow-cli/src/fetch.rs:20-24, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no cap and no deadline | A large package or a slow mirror never fails (reasoned from crates/meow-cli/src/fetch.rs:18-25, low) | REQ-1819 requires a deadline, and a gigabyte arriving as a package would fill a disk (from docs/requirements/REQ-1819-fetch-carries-deadline.md and crates/meow-cli/src/fetch.rs:20-24, high) |
| Cap the bytes as they stream in | A huge response is stopped before it fills memory (reasoned from crates/meow-cli/src/fetch.rs:122-138, low) | Not weighed in #162, which reads the whole body with `response.bytes()` and measures it after (from crates/meow-cli/src/fetch.rs:122-128, high) |

## What it costs

A package over 64 MiB can't be used, and a download over a slow link that takes
longer than 120 seconds fails (reasoned from crates/meow-cli/src/fetch.rs:18-25,
low). A response larger than the cap is held in memory in full before it is
refused (from crates/meow-cli/src/fetch.rs:122-138, high).

## What would reverse it

- A published package that a workspace needs exceeds 64 MiB, or its mirror
  can't serve it in 120 seconds (reasoned from
  crates/meow-cli/src/fetch.rs:18-25, low).

## Consequences

- The timeout covers the whole request, connecting included, because it is
  `reqwest`'s `timeout` on the client (from crates/meow-cli/src/fetch.rs:97-100,
  high).

## How I will know it was realised

1. `TIMEOUT` is 120 seconds and `MAX_BYTES` is 64 MiB in
   crates/meow-cli/src/fetch.rs (from crates/meow-cli/src/fetch.rs:18 and :25,
   high). No test in crates/meow-cli/tests/fetch.rs exercises either; one
   serving 65 MiB and one that stalls would.

## What this does not settle

- Whether the cap applies to the archive or to what it unpacks to: a small
  archive that decompresses past 64 MiB passes (reasoned from
  crates/meow-cli/src/fetch.rs:130-138 and :190-200, low).
