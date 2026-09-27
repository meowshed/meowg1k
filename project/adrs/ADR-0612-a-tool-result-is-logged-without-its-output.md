---
id: ADR-0612
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2264]
supersedes: []
---

# 0612. A tool result is logged without the tool's output

## Decision

The session log records each tool result with its call's identifier, its
duration and its error, and with an empty output (from
https://github.com/retran/meowg1k/pull/129 and
crates/meow-star/src/run.rs:722-733, high). The event records that the call
happened and how it ended, not what it produced (from
https://github.com/retran/meowg1k/pull/129, high).

Once this is accepted, `meow session show` and export report a failed call's
error and nothing a call returned (from
crates/meow-session/src/export.rs:152-156, high).

## Why

The engine's `AgentEvent::ToolEnd` carries the duration and the error but not
the output, and carrying it means widening that event or reading it back from
the message list, which #129 left to be decided deliberately (from
https://github.com/retran/meowg1k/pull/129 and
crates/meow-agent/src/event.rs:46-53, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: write no `ToolResult` event | The log doesn't hold an event whose `output` is always empty (reasoned from crates/meow-star/src/run.rs:728, low) | The log would not show that a call ended, how long it took or why it failed, and REQ-2203 lists `ToolResult` among the kinds (from docs/requirements/REQ-2203-event-kinds-are-fixed.md, medium) |
| Widen `AgentEvent::ToolEnd` to carry the output | Every consumer of the engine's events, the log included, gets what the tool returned (from https://github.com/retran/meowg1k/pull/129, high) | Not chosen in #129, which left the change to its own decision (from https://github.com/retran/meowg1k/pull/129, high) |
| Read the output back from the message list | No change to the engine's event (from https://github.com/retran/meowg1k/pull/129, high) | Not chosen in #129, for the same reason (from https://github.com/retran/meowg1k/pull/129, high) |

## What it costs

Nobody can see afterwards what a tool returned, so an audit can say a file was
read and not what the model saw of it (reasoned from
crates/meow-star/src/run.rs:728-733, low). The design gives `ToolResult` an
`output` blob, which the log leaves empty (from docs/design/0.3.0-sessions.md
section 3, high).

## What would reverse it

- A requirement asks export, audit or resume to show what a tool returned
  (reasoned from https://github.com/retran/meowg1k/pull/129, low).

## Consequences

- A resumed run rebuilds only user, assistant and compaction messages, so it
  doesn't need tool output today (from crates/meow-cli/src/session.rs:97-116,
  high).

## How I will know it was realised

1. `Relay` in crates/meow-star/src/run.rs records `ToolResult` with
   `output: String::new()` (from crates/meow-star/src/run.rs:728-733, high). No
   test asserts it.

## What this does not settle

- Which of the two ways to carry the output, if either, a later change takes
  (from https://github.com/retran/meowg1k/pull/129, high).
