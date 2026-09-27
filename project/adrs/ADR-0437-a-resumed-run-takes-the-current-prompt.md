---
id: ADR-0437
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2238]
supersedes: []
---

# 0437. A resumed run takes its system prompt from the specification, not the history

## Decision

The caller supplies the message list, and the system prompt still comes from the
specification rather than from the history (from
https://github.com/retran/meowg1k/pull/129, high).

Once this is accepted, `Engine::resume` puts the agent specification's system
prompt first, drops every system message from the history it was given, and
appends the new task (from crates/meow-agent/src/engine.rs:76, high). The binary
rebuilds the history from `Sessions::events_for_model`, so a compacted range
arrives as its summary (from crates/meow-cli/src/session.rs:97, high). The
rebuild keeps only user and assistant text, so the tool calls and results of the
earlier run don't reach the resumed one (from
crates/meow-cli/src/session.rs:100, high).

## Why

An agent whose prompt changed between runs then runs under the new one, which is
most of why anybody resumes with a different model (from
https://github.com/retran/meowg1k/pull/129, high). Rebuilding the list means
reading a log, and the engine doesn't know the store exists (from
https://github.com/retran/meowg1k/pull/129, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Take the system prompt from the session's history | A resumed run would repeat the original exactly, prompt and all (reasoned from crates/meow-agent/src/engine.rs:73, low) | An agent whose prompt changed would resume under the old one (from https://github.com/retran/meowg1k/pull/129, high) |
| Let the engine rebuild the message list from the log | One call would resume a session, with no rebuild in the binary (reasoned from crates/meow-cli/src/session.rs:97, low) | The engine doesn't know the store exists, and rebuilding means reading a log (from crates/meow-agent/src/engine.rs:69, high) |
| Do nothing: no resume path in the engine | The engine keeps one entry point, `run`, which is now a `resume` with an empty history (from crates/meow-agent/src/engine.rs:62, high) | `--continue` needs the earlier conversation, and REQ-2238 requires the resumed run to see compaction as the original did (from docs/spec/session.md [R-SESSION-051], high) |

## What it costs

A resumed run doesn't reproduce the original run's conditions: if the agent's
prompt changed, the earlier answers were written under a prompt the model no
longer sees (reasoned from crates/meow-agent/src/engine.rs:86, low). The log
can't say which prompt the original ran under, because `Started` records the
task and the agent and not the prompt (from crates/meow-core/src/event.rs:75,
high).

## What would reverse it

If the log began to record the system prompt, and resuming had to reproduce a
run exactly, for example to replay it for an audit, the history's prompt would
have to win (reasoned from crates/meow-core/src/event.rs:75, low).

## Consequences

- `Engine::run` is `resume` with an empty history, so a fresh run and a resumed
  one go through one loop (from crates/meow-agent/src/engine.rs:62, high).
- `meow-star` passes the session's history to the engine on every `agent.run`,
  and an empty history is a fresh run (from crates/meow-star/src/run.rs:306,
  high).
- A system message in the supplied history is dropped, so a caller can't
  smuggle a second system prompt in (from crates/meow-agent/src/engine.rs:89,
  high).

## How I will know it was realised

`a_resumed_run_sees_what_the_original_saw` in
`crates/meow-session/tests/resume.rs` passes, which covers the rebuild with
compaction (from crates/meow-session/tests/resume.rs:59, high). No test yet
checks that a resumed run's first message is the specification's current
prompt; one would call `Engine::resume` with a history holding a different
system message and assert which one the provider received (reasoned from
crates/meow-agent/src/engine.rs:86, low).

## What this does not settle

- Whether the rebuild should carry tool calls and tool results: today it keeps
  user and assistant text only (from crates/meow-cli/src/session.rs:100, high).
- Whether a handler that calls `agent.run` twice in one resumed run gives both
  agents the resumed history (from crates/meow-star/src/run.rs:306, medium).
