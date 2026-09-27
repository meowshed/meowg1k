---
id: BUG-0115
artifact: bug
status: approved
severity: critical
violates: REQ-2043
found: 2026-09-27
revised: 2026-09-27
issue: 238
---

# A sensitive argument is written to the session log in the clear

A policy rule can mark an argument sensitive, and the approval prompt redacts
it, but the `ToolCall` event is written with the model's raw arguments, so the
value sits unredacted in `.meow/.data/meow.db`.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare a tool `login` with an argument `token`, and
   `meow.policy(rules = [{"tools": ["login"], "decision": "allow", "sensitive": ["token"]}])`.
2. Run an agent whose scripted model calls `login` with
   `{"token": "hunter2"}`.
3. Run `sqlite3 .meow/.data/meow.db "select body from events"`. A row holds
   `hunter2`.

This follows from the code below.

## What the system does

The engine sends `AgentEvent::ToolStart` with `args: call.arguments.clone()`,
`crates/meow-agent/src/engine.rs:285-290`, and `Relay` records it as
`EventKind::ToolCall { args }` unchanged, `crates/meow-star/src/run.rs:707-712`.
Redaction runs only when building the approval prompt,
`crates/meow-policy/src/prompt.rs:63-69`.

## What it should do, and why

REQ-2043: "A value marked sensitive MUST NOT be written to the session log in
the clear." REQ-2042 asks the same for the transcript. The arguments should be
redacted with `meow_policy::redact` and `Policy::sensitive_for` before the
event is sent to any sink.

## Triage

The defect enters in the engine, which emits the raw arguments, and in
`meow-star`, which records them. It's critical because the vision's first goal
is security, and a secret a person marked sensitive is kept on disk in a file
the index, exports and backups all reach.

## Closed by

A test in `crates/meow-star/tests/approval.rs`, named for example
`a_sensitive_argument_is_redacted_in_the_log`, that runs the reproduction above
and expects no event body to contain the value.
