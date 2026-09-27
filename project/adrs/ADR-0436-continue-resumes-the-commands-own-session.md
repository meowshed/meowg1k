---
id: ADR-0436
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2850, REQ-2851, REQ-2245]
supersedes: []
---

# 0436. `--continue` resumes the invoked command's own session and never starts a fresh one

## Decision

`--continue` picks the invoked command's own most recent session rather than the
newest one in the workspace, and with nothing of that command's to continue it
fails and says so (from https://github.com/meowshed/meowg1k/pull/129, high).

Once this is accepted, `open_session` looks up the newest session whose agent is
the invoked command and resumes it, and with none it prints "`--continue` found
no earlier session of `<agent>` in this workspace" and exits 2 before the run
starts (from crates/meow-cli/src/wire.rs:1184 and
crates/meow-cli/src/exit.rs:42, high). A specific older session can't be
resumed: the design's `meow session resume <id>` has no subcommand in the
binary, which offers `list`, `show`, `fork`, `export` and `gc` (from
docs/design/0.3.0-sessions.md section 6 and
https://github.com/meowshed/meowg1k/pull/129, high).

## Why

Picking the newest session anywhere would let continuing one agent resume
another's run (from https://github.com/meowshed/meowg1k/pull/129, high). A flag
that quietly started a fresh run would give somebody a conversation they thought
they were adding to (from https://github.com/meowshed/meowg1k/pull/129, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Resume the newest session in the workspace | One lookup with no agent filter, and it continues whatever ran last whichever command it was (reasoned from crates/meow-store/src/rows.rs:266, low) | Continuing one agent could resume another's run (from https://github.com/meowshed/meowg1k/pull/129, high) |
| Start a fresh run when there is nothing to continue | A script could pass `--continue` on every call, including the first, without checking whether a session exists (reasoned from crates/meow-cli/tests/surface.rs:374, low) | Somebody gets a conversation they thought they were adding to (from https://github.com/meowshed/meowg1k/pull/129, high) |
| Do nothing: every run is fresh and there is no `--continue` | A run never picks up context it wasn't given, which is why fresh stays the default (from docs/design/0.3.0-sessions.md section 6, high) | A failed run is a dead end with no resume, which the design lists as a gap in v0.2.x (from docs/design/0.3.0-sessions.md section 2, high) |

## What it costs

A script that passes `--continue` on its first call fails with exit 2, so it has
to know whether the command has run before (from
crates/meow-cli/tests/surface.rs:374, high). A user who wants another
command's session or an older one of the same command has no flag for it, and
can only fork it with `meow session fork` (reasoned from
crates/meow-cli/src/wire.rs:1184, low).

## What would reverse it

A command that shares its session with another command, such as a follow-up
verb meant to continue `review`, would need a lookup wider than the invoked
command's own sessions (reasoned from crates/meow-cli/src/session.rs:159, low).
Amending REQ-2850 or REQ-2851 would also reverse it.

## Consequences

- The session is opened before the run, so a failed `--continue` runs nothing
  and writes nothing (from crates/meow-cli/src/wire.rs:1092, high).
- A missing session exits 2, as a usage mistake, and a store that won't open
  exits 7, so a script can tell the two apart (from
  crates/meow-cli/src/wire.rs:1094, high).
- `Log::open` takes `resuming` as an argument, so the session layer never
  decides on its own to continue, as REQ-2245 asks (from
  crates/meow-cli/src/session.rs:40, high).

## How I will know it was realised

Three tests in `crates/meow-cli/tests/surface.rs` pass:
`continue_with_nothing_to_continue_fails`,
`continue_adds_to_the_most_recent_session_of_that_command` and
`continue_does_not_resume_another_commands_session` (from
crates/meow-cli/tests/surface.rs:374, high).

## What this does not settle

- "Most recent" means the newest by identifier, that is by creation time, so a
  resumed older session doesn't become the one `--continue` picks next (from
  crates/meow-store/src/rows.rs:266, high).
- Whether a sub-agent's session, which carries its own agent name and a parent,
  can be picked when a command of that name is invoked: the lookup doesn't
  filter on the parent (from crates/meow-store/src/rows.rs:269, medium).
- Resuming a session by identifier, which the design shows as
  `meow session resume` and the binary doesn't offer (from
  docs/design/0.3.0-sessions.md section 6, high).
