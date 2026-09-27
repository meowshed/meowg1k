---
id: BUG-0118
artifact: bug
status: approved
severity: major
violates: REQ-2238
found: 2026-09-27
revised: 2026-09-27
issue: 241
---

# A resumed run loses the earlier run's tool calls and results

Rebuilding a session's history for `--continue` keeps user text, assistant text
and compaction summaries, and drops every tool call, tool result and assistant
turn whose text was empty, so the resumed model sees a different conversation
from the one the original saw.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run an agent whose scripted model calls `read_file` on `a.rs`, then answers
   "a.rs defines main".
2. Run the same command with `--continue` and capture the request the model
   receives.
3. The request holds the first task and "a.rs defines main", and no tool call
   or tool result.

This follows from the code below.

## What the system does

`rebuild` in `crates/meow-cli/src/session.rs:97-117` matches `Started`,
`UserMessage`, a non-empty `Assistant` and `Compaction`, and ignores every
other kind with `_ => {}`. `ToolCall` and `ToolResult` fall there, and so does
an `Assistant` turn that held only tool calls. ADR-0437 records the loss.

## What it should do, and why

REQ-2238: "Resuming MUST rebuild the message list per [R-SESSION-011], so a
resumed run sees compaction exactly as the original did." REQ-2211 describes
that rebuild as skipping superseded ranges and substituting summaries, and
nothing else. The rebuild should keep the tool calls on their assistant
messages and the tool results as tool messages.

## Triage

The defect enters in `meow-cli`'s `rebuild`. It's major because resuming is
there to continue work the agent has done, and the evidence it gathered with
tools is exactly what goes missing; the model then repeats the calls or
answers without them.

## Closed by

A test in `crates/meow-cli/tests/`, named for example
`a_resumed_run_sees_the_earlier_tool_results`, that records a run with one tool
call and expects the rebuilt history to hold the call and its result in order.
