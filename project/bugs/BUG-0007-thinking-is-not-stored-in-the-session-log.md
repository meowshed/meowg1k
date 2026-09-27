---
id: BUG-0007
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue:
---

# Thinking is not stored in the session log

The engine keeps a response's thinking on the in-memory assistant message, and
nothing writes it to the session log. `EventKind::Assistant` has no field for
it, and nothing records it as a `Note` with level `thinking` either, so
`meow session export --thinking` has nothing to include and a resumed session
has no thinking to send back.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run `grep -rn '"thinking"' crates`. Outside `meow-llm`, the only writer of a
   `Note` with level `thinking` is the test `crates/meow-session/tests/fork.rs:369`.
2. Read `EventKind::Assistant` in `crates/meow-core/src/event.rs:86-92`: it holds
   `content` and `tool_calls` only.

## What the system does

`crates/meow-agent/src/engine.rs:270-273` copies `response.thinking` onto the
message kept in the run's history. The relay in
`crates/meow-star/src/run.rs:640-647` records `Assistant { content, tool_calls }`
and sends `ThinkingDelta` only to the live region
(`crates/meow-star/src/run.rs:698`). The export filter in
`crates/meow-session/src/export.rs:206-216` looks for `Note { level:
"thinking" }`, which nothing in production writes.

## What it should do, and why

No requirement says the session log stores thinking. REQ-1623 keeps thinking on
the assistant message, and REQ-2261 has export omit it unless asked, which
presumes the log holds it. ADR-1602, "thinking is streamed and stored", gives
the reason: a provider can require thinking on a later turn, and a resumed run
can only send what the log kept. This is a gap in the requirements and routes to
the requirements step.

## Triage

The defect enters in the relay that turns engine events into log events. It is
minor today, because no provider request enables extended thinking, so
Anthropic never returns any; an OpenAI-shaped server that sends
`reasoning_content` does reach this path.

## Closed by

A test in `crates/meow-star/tests/`, named for example
`thinking_reaches_the_session_log`, that runs an agent against a recorded
response carrying thinking and expects the log, and an export with
`include_thinking`, to hold it.
