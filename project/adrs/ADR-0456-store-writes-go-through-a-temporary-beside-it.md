---
id: ADR-0456
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1207, REQ-1208]
supersedes: []
---

# 0456. A store write goes to a temporary file beside the target, then a rename

## Decision

Writing is a temporary file beside the target, then a rename (from
https://github.com/meowshed/meowg1k/pull/153, high).

Once this is accepted, `write_json` writes the whole file to `<target>.new`
with mode `0600`, calls `sync_all` on it and renames it over the target (from
crates/meow-cli/src/auth.rs:234 and crates/meow-cli/src/auth.rs:302, high). The
credential store and the trust record both go through it (from
crates/meow-cli/src/auth.rs:211 and crates/meow-cli/src/trust.rs:214, high).
The directory isn't synced after the rename, so the rename itself may not
survive a power loss (reasoned from crates/meow-cli/src/auth.rs:254, low).

## Why

A rename across filesystems isn't atomic, so the temporary file goes beside the
target rather than in `/tmp` (from https://github.com/meowshed/meowg1k/pull/153,
high). A truncated credential store locks the user out of every provider at once
(from https://github.com/meowshed/meowg1k/pull/153, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| A temporary file in `/tmp` | Nothing extra is left in `~/.meow/` when a write dies halfway (reasoned from crates/meow-cli/src/auth.rs:253, low) | A rename across filesystems isn't atomic, and `/tmp` is often a different one (from crates/meow-cli/src/auth.rs:227, high) |
| Do nothing: write the file in place, as v0.2.x's `persistCopilotToken` did with `os.WriteFile` | One call and no temporary file (from `git show v0.2.1:cmd/auth.go`, medium) | A process that dies mid-write leaves a truncated file, and a truncated store locks the user out of every provider at once (from docs/spec/auth.md [R-AUTH-012], high) |

The code and the history name no third option (from
https://github.com/meowshed/meowg1k/pull/153, medium).

## What it costs

A write that dies leaves `auth.json.new` beside the store until the next write
truncates and reuses it (from crates/meow-cli/tests/auth.rs:244, high). The
temporary has a fixed name, so two processes writing the store at once share one
temporary and the last rename wins, losing the other's entry (reasoned from
crates/meow-cli/src/auth.rs:253, low).

## What would reverse it

Moving credentials to the operating system's secret store, which ADR-0467
proposes, would leave the file only as a fallback and make this write path
matter only there (from
docs/adrs/ADR-0467-credentials-live-in-the-os-secret-store.md, medium).

## Consequences

- A process that dies mid-write leaves the old store whole (from
  crates/meow-cli/src/auth.rs:199, high).
- The trust record gets the same protection, so a truncated trust file doesn't
  ask every question again (from crates/meow-cli/src/auth.rs:222, high).
- On Windows the temporary is written with `std::fs::write` and no mode, so it
  inherits the user's ACL (from crates/meow-cli/src/auth.rs:326, high).

## How I will know it was realised

`a_failed_write_does_not_truncate_what_was_there` in
`crates/meow-cli/tests/auth.rs` passes: with a truncated `auth.json.new` left
behind, the next login ends with valid JSON holding both credentials (from
crates/meow-cli/tests/auth.rs:232, high). The test simulates a leftover
temporary and doesn't kill a process mid-write (from
crates/meow-cli/tests/auth.rs:244, high).

## What this does not settle

- Concurrent writers: nothing locks the store, and the temporary's name is fixed
  (from crates/meow-cli/src/auth.rs:253, high).
- Durability after a power loss, which would need the directory synced after
  the rename (reasoned from crates/meow-cli/src/auth.rs:254, low).
