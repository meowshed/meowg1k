---
id: ADR-1006
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1039, REQ-1040, REQ-1041, REQ-1042, REQ-1043]
supersedes: []
---

# 1006. Compaction runs in the engine, not in a userland library

## Decision

The engine compacts before the next model call when the rebuilt message list
exceeds the configured fraction of the context window. (from docs/spec/agent.md
[R-AGENT-040], high)

## Why

In v0.2.x compaction lives in a userland `.star` library that each command has
to remember to call. (from docs/spec/agent.md Changes from v0.2.x, high)

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: a userland `.star` library, as v0.2.x had | A workspace can change how it compacts without changing the binary (reasoned from docs/design/0.3.0-architecture.md section 2, low) | Each command has to remember to call it (from docs/spec/agent.md Changes from v0.2.x, high), and the library marked the replaced events obsolete, so a compacted run could no longer be replayed in full (from docs/design/0.3.0-sessions.md section 2, high) |
| Compact after the provider rejects an over-long context | No summary call until one is needed (reasoned from crates/meow-agent/src/engine.rs:161-167, low) | The engine acts before the next call, not after a provider rejects one (from crates/meow-agent/src/engine.rs:161-167, high) |

The specification and the history name no third place for compaction to run.

## What it costs

The engine owns a model call it didn't make before, and a token estimate of four
characters to a token that decides when to make it (from
crates/meow-agent/src/compaction.rs:37-48 and
https://github.com/meowshed/meowg1k/pull/120, high). `meow-agent` emits the
compaction and doesn't write it, so whoever persists has to turn the event into
the session record (from https://github.com/meowshed/meowg1k/pull/120, high).

## What would reverse it

- A workspace needs a compaction strategy that the three settings, `at`,
  `keep_recent` and `model`, can't express (reasoned from
  crates/meow-agent/src/compaction.rs:8-35, low).

## Consequences

A declaration sets compaction with `compaction =` on the agent, and the default
summarises at 80% of the context and keeps the last 10 messages (from
docs/design/0.3.0-starlark-api.md section 5.1 and
crates/meow-agent/src/compaction.rs:27-35, high). `compaction.star` is deleted
with the other userland libraries, roughly 2,500 lines, which the architecture
calls the clearest single measure of whether the redesign is worth doing (from
docs/design/0.3.0-architecture.md section 10, high). The Starlark runtime
records each compaction as a `Compaction` session event (from
crates/meow-star/src/run.rs:757-768, high).

## How I will know it was realised

1. These tests in crates/meow-agent/tests/spec.rs pass:
   `compaction_triggers_on_the_threshold_and_keeps_the_recent_ones`,
   `compaction_summarises_and_records_what_it_replaced` and
   `a_failed_compaction_fails_the_run_rather_than_sending_an_over_long_context`
   (from crates/meow-agent/tests/spec.rs:842-944, high).
2. No file under `.meow/` in this repository calls a compaction function (from
   https://github.com/meowshed/meowg1k/pull/137, high).

## What this does not settle

- Whether compaction summarises or drops, which ADR-1001 settles.
- Which model summarises, which ADR-1002 settles.
- How a resumed run sees a compaction, which the session log's rebuild settles
  (from crates/meow-agent/src/engine.rs:68-70, high).
