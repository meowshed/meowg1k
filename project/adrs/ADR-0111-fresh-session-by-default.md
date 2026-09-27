---
id: ADR-0111
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2244, REQ-2245]
supersedes: []
---

# 0111. A run starts a fresh session unless it names one

## Decision

Fresh, with no history, is the default, and the source names it as a choice in
place of an absence (from docs/design/0.3.0-sessions.md section 6, high).
`Log::open` takes the session to resume as an argument and starts a new one
when it gets none, so nothing in the session layer works out a continuation
from the workspace (from crates/meow-cli/src/session.rs:40-73, high).

## Why

An automation invoked from CI shouldn't pick up yesterday's context because the
two runs happen to share a workspace (from docs/design/0.3.0-sessions.md section
6, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no resume at all, as in v0.2.x | Every run is fresh without any flag to get wrong, and there is no resume path to build (reasoned from docs/design/0.3.0-sessions.md section 2, low) | A failed run is a dead end, and a follow-up has to re-establish its context from nothing (from docs/design/0.3.0-sessions.md sections 1 and 2, high) |
| Continue from the workspace's earlier context without being asked | A follow-up needs no flag and no session identifier (reasoned from docs/design/0.3.0-sessions.md section 1 question 2, low) | A CI run would pick up yesterday's context because it shares a workspace (from docs/design/0.3.0-sessions.md section 6, high) |
| `--continue` resumes the newest session in the workspace, whatever command wrote it | One flag reaches the last thing that ran, which is shorter to explain (reasoned from https://github.com/meowshed/meowg1k/pull/129, low) | Continuing one agent could resume another's run, so the flag picks the invoked command's own session (from https://github.com/meowshed/meowg1k/pull/129, high) |

## What it costs

A person who wants a follow-up has to say so with `--continue` or
`meow session resume`, and `--continue` fails when the command has no session
of its own in place of quietly starting fresh (from docs/philosophy.md section
2, high). The resume path needs the caller to rebuild the message list, because
the engine doesn't know the store exists (from
https://github.com/meowshed/meowg1k/pull/129, high).

## What would reverse it

- A requirement that a run pick up earlier context without naming a session,
  which would contradict REQ-2245 and the principle that a run is a task and
  not a conversation (reasoned from docs/philosophy.md section 2, low).

## Consequences

Continuing is explicit and narrow: `--continue` resumes the most recent session
of the command being invoked, and fails when that command has none (from
docs/philosophy.md section 2, high). The engine's resume path takes an empty
history as a fresh run, so the Starlark runtime never infers a continuation
either (from crates/meow-star/src/run.rs:306-309, high).

## How I will know it was realised

1. `a_run_that_names_no_session_starts_one` in
   crates/meow-session/tests/resume.rs passes: two runs of one command leave
   two sessions (from crates/meow-session/tests/resume.rs:103, high).
2. `continue_adds_to_the_most_recent_session_of_that_command`,
   `continue_with_nothing_to_continue_fails` and
   `continue_does_not_resume_another_commands_session` in
   crates/meow-cli/tests/surface.rs pass against the built binary (from
   crates/meow-cli/tests/surface.rs:371-430, high).

## What this does not settle

- Whether session state set with `ctx.session.set` is replayed on resume: it is
  written as a `Note` and read back from memory within the run only, and making
  it replayable needs a new event kind (from
  https://github.com/meowshed/meowg1k/pull/129, high).
- Forking, which starts a new session from a prefix of an old one, is its own
  operation with its own requirements (from docs/design/0.3.0-sessions.md
  section 6, high).
