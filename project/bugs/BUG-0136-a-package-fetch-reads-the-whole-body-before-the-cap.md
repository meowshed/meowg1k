---
id: BUG-0136
artifact: bug
status: approved
severity: minor
violates: REQ-1828
found: 2026-09-27
revised: 2026-09-27
issue: 259
---

# A package fetch reads the whole body into memory before checking the cap

`fetch` reads the response body with `response.bytes()` and only then compares
its length with 64 MiB, so a source that sends gigabytes is held in memory
before it's refused.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Serve an endless stream of bytes at a local URL and declare a package whose
   source is that URL.
2. Run `meow pkg update` and watch its memory. It grows until the 120-second
   timeout, the end of the stream or the machine stops it.

This follows from the code below; it wasn't run.

## What the system does

`crates/meow-cli/src/fetch.rs:121-127` awaits `response.bytes()`, and
`fetch.rs:130-138` checks `bytes.len() > MAX_BYTES` afterwards. The comment on
`MAX_BYTES`, `fetch.rs:20-24`, says refusing is "cheaper than filling a disk";
ADR-0622 records that the cap protects the disk and not the memory.

## What it should do, and why

REQ-1828: "A fetch MUST refuse an archive larger than 64 MiB." Refusing after
reading it all is refusing to keep it, not refusing it. The fetch should check
`Content-Length` when present and stop reading once the running total passes
the cap.

## Triage

The defect enters in `fetch`. It's minor: the archive is refused and nothing
reaches the disk, and the 120-second timeout bounds how much arrives. With
BUG-0125, an untrusted workspace can trigger it.

## Closed by

A test in `crates/meow-cli/tests/fetch.rs`, named for example
`a_fetch_stops_reading_past_the_cap`, that serves 65 MiB and counts the bytes
the server managed to send before the connection closed, expecting well under
the full body.
