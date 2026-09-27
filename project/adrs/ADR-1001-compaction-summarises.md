---
id: ADR-1001
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1044, REQ-1045, REQ-1046]
supersedes: []
---

# 1001. Compaction summarises the superseded range with a model call

## Decision

Compaction summarises the superseded range with a model call, records the tokens
the summary saved, and never drops a message it didn't summarise. (from
docs/spec/agent.md [R-AGENT-044], high)

## Why

The alternative is losing information without saying so, and a silent drop
leaves the model confidently wrong about what it already knows. Recording the
tokens saved makes the cost of the summary visible. (from docs/spec/agent.md
Decisions and [R-AGENT-044], high)

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: never compact, and send the whole history on every call | Costs no model call and loses nothing from the context (reasoned from docs/spec/agent.md [R-AGENT-043], low) | A long run grows past the context window, and the provider rejects an over-long context (from docs/spec/agent.md [R-AGENT-043], high) |
| Drop superseded messages without summarising them | It costs no model call (from git show f8e58ea:docs/spec/agent.md Open questions, high) | It loses information without saying so, and leaves the model confidently wrong about what it already knows (from docs/spec/agent.md Decisions and [R-AGENT-044], high) |

The first draft of the specification framed the question as summarise or drop
and named no third way to shrink a context (from git show
f8e58ea:docs/spec/agent.md Open questions, high).

## What it costs

Compaction costs a model call at the moment the run is already long. (from
docs/spec/agent.md Decisions, high) The engine sends the summary request with
a limit of 1,024 output tokens, and a summary of a short exchange can be longer
than what it replaces, in which case the recorded saving is zero (from
crates/meow-agent/src/engine.rs:186-215 and
https://github.com/meowshed/meowg1k/pull/120, high).

## What would reverse it

- The `tokens_saved` recorded on `Compaction` events across real runs is
  routinely zero, which would mean the summary call costs more than it saves
  (reasoned from crates/meow-agent/src/engine.rs:212-221, low).

## Consequences

The engine emits an `AgentEvent::Compacted` carrying the superseded range, the
summary and `tokens_saved`, and replaces the range in its message list with one
message headed "Summary of the earlier conversation" (from
crates/meow-agent/src/engine.rs:208-223, high). The Starlark runtime turns that
event into a `Compaction` session event, so the saving is in the log (from
crates/meow-star/src/run.rs:757-768, high). The engine skips compaction when
fewer than two messages would be summarised, because the call would cost more
than it saves (from crates/meow-agent/src/compaction.rs:50-66, high).

## How I will know it was realised

1. `compaction_summarises_and_records_what_it_replaced` in
   crates/meow-agent/tests/spec.rs passes (from
   crates/meow-agent/tests/spec.rs:869, high).

## What this does not settle

- How tokens are counted. The engine estimates four characters to a token,
  which is close enough for a threshold and wrong enough that nothing should
  bill from it (from crates/meow-agent/src/compaction.rs:37-48 and
  https://github.com/meowshed/meowg1k/pull/120, high).
- Which model summarises, which ADR-1002 settles.
- When compaction fires, which ADR-1006 settles.
