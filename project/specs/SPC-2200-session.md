---
id: SPC-2200
artifact: spec
status: live
revised: 2026-09-27
checked-at:
states: [REQ-2200, REQ-2201, REQ-2202, REQ-2203, REQ-2204, REQ-2205, REQ-2206, REQ-2207, REQ-2208, REQ-2209, REQ-2210, REQ-2211, REQ-2212, REQ-2213, REQ-2214, REQ-2215, REQ-2216, REQ-2217, REQ-2218, REQ-2219, REQ-2220, REQ-2221, REQ-2222, REQ-2223, REQ-2224, REQ-2225, REQ-2226, REQ-2227, REQ-2228, REQ-2229, REQ-2230, REQ-2231, REQ-2232, REQ-2233, REQ-2234, REQ-2235, REQ-2236, REQ-2237, REQ-2238, REQ-2239, REQ-2240, REQ-2241, REQ-2242, REQ-2243, REQ-2244, REQ-2245, REQ-2246, REQ-2247, REQ-2248, REQ-2249, REQ-2250, REQ-2251, REQ-2252, REQ-2253, REQ-2254, REQ-2255, REQ-2256, REQ-2257, REQ-2258, REQ-2259, REQ-2260, REQ-2261, REQ-2262, REQ-2263, REQ-2264]
---

# The session log of an agent run

## Scope

`meow-session` owns the durable record of an agent run: the append-only event
log, the identifiers people type, the lifecycle, and the operations that
continue or branch a run. It sits on `meow-store` and knows nothing about the
engine that produces the events (from docs/spec/session.md, high).

It doesn't decide when to compact, what a budget is or how a transcript looks.
It records what happened and replays it two ways: for a model and for a person
(from docs/spec/session.md, high).

## Boundary

| Surface | What it is |
| --- | --- |
| The `meow-session` Rust API | The crate's public interface to the session log (from docs/spec/session.md, high) |
| Rows in `meow-store` | What the session layer writes through `meow-store` (from docs/spec/session.md, high) |
| `meow session` commands | The identifiers and JSON that reach a user (from docs/spec/session.md, high) |

## Behaviour

### The log

The event log is append-only [REQ-2200]: the session layer never updates or
deletes an event it has written [REQ-2201]. Each event carries a sequence
number, monotonic and gapless within its session and starting at 1 [REQ-2202],
and the UTC timestamp at which it was recorded [REQ-2204].

The event kinds are exactly `Started`, `UserMessage`, `Assistant`, `ToolCall`,
`ToolResult`, `Policy`, `Usage`, `Compaction`, `Note` and `Finished` [REQ-2203].
A `ToolResult` event can carry an empty output, recording the call's
identifier, duration and error without what the tool returned [REQ-2264].

A session holds one or more runs [REQ-2205]. Each run begins with a `Started`
event [REQ-2206] and, once it has ended, is closed by exactly one `Finished`
event before the next `Started` event [REQ-2207]. Resuming a session appends a
new `Started` event [REQ-2208].

### Compaction

A `Compaction` event names the inclusive sequence range it supersedes [REQ-2209]
and leaves the events in that range in place [REQ-2210]. A `Compaction` event
never supersedes a range an earlier one already supersedes [REQ-2214].

Rebuilding the message list for a model call skips every superseded range and
substitutes that range's summary [REQ-2211]. Rebuilding the log for display or
export returns the original events [REQ-2212] with no summary substituted
[REQ-2213].

### Usage and cost

A `Usage` event carries prompt tokens, completion tokens, cached prompt tokens
and cost as separate typed fields [REQ-2215]. The cost is computed when the
event is written, from the price table in effect at that moment [REQ-2216], and
isn't recomputed on read [REQ-2217]. A session reports its totals as the sum of
its own `Usage` events plus the totals of its child sessions [REQ-2219].

A call against a model that declares a price records its cost from the usage the
provider returned: prompt tokens at the input price plus completion tokens at
the output price, each price per million tokens [REQ-2262].

### Identity

Every session has a full identifier that sorts by creation time, and a short
identifier that is its last eight characters [REQ-2220].

The selectors `@last`, `@last-N` and `@<agent-name>` resolve at the moment of
use: `@last` to the most recent session in the workspace, `@last-N` to the Nth
most recent, and `@<agent-name>` to the most recent session of that agent
[REQ-2223].

A session can carry a name [REQ-2224]. A name is unique within the workspace
[REQ-2225], doesn't begin with `@` [REQ-2226] and resolves like an identifier
[REQ-2227].

### Lifecycle

A session is in exactly one state: `running`, or one of the terminal states
`finished`, `budget`, `cancelled`, `denied`, `tool_aborted` or `failed`
[REQ-2228]. The log defines the state: `running` when the last event isn't
`Finished`, and otherwise the stop reason that last `Finished` event carries
[REQ-2229]. A denormalised copy of the state can be kept, provided it is
rebuildable from the log and the log wins on any disagreement [REQ-2230].

While a run is in flight, its writer updates a heartbeat timestamp on the
session row at a fixed interval [REQ-2231]. The heartbeat isn't an event
[REQ-2232].

### Continuing and branching

Resuming a session appends to it, continuing the existing sequence [REQ-2236],
and creates no new session [REQ-2237]. Resuming rebuilds the message list with
compaction applied as for a model call [REQ-2238].

Forking at sequence `n` creates a new session whose first `n` events are copies
referencing the same blobs [REQ-2239], increments the reference count of every
blob so referenced [REQ-2240], records the origin session and sequence
[REQ-2241] and leaves the origin unchanged [REQ-2242].

A run that names no session starts a fresh one [REQ-2244]; the session layer
doesn't infer continuation from the workspace [REQ-2245].

### Parentage

A session started by a sub-agent records its parent [REQ-2246], and the parent
can enumerate its children in creation order [REQ-2247]. A session's parent
already exists when the session is created [REQ-2249], so the parent relation
has no cycle [REQ-2248].

### Audit

Every tool invocation produces a `Policy` event before the tool runs, recording
the decision, the rule that produced it, and whether the decision came from a
rule, an interactive answer or a session grant [REQ-2250]. A `Policy` event is
written for denied invocations as well as allowed ones [REQ-2251].

### Retention

Garbage collection deletes whole sessions only [REQ-2252], and deletes a parent
only together with its descendants [REQ-2253]. It keeps a named session unless
the caller asks for named sessions explicitly [REQ-2254].

Retention is configurable by age, by count and by total database size
[REQ-2255], and applies the strictest of the configured limits [REQ-2256].

### Export

JSON export uses the schema definition and version number that the live
`--format json` renderer uses [REQ-2257], and every persisted event kind
serialises identically in both [REQ-2258].

Markdown export includes the transcript, the tool calls with their policy
decisions, and the usage totals [REQ-2259].

Export in both formats redacts every value the policy marked sensitive
[REQ-2260], and omits thinking content unless it is asked for explicitly
[REQ-2261].

## Failure paths

| Condition | What happens |
| --- | --- |
| A model has no price in the price table | The `Usage` event records the cost as absent, never as zero [REQ-2218] |
| A short identifier matches more than one session | Resolving fails with an error listing the candidates [REQ-2221] and picks none [REQ-2222] |
| The last event isn't `Finished` and the heartbeat is older than three intervals | The session is treated as dead [REQ-2233] |
| A session is opened whose last event isn't `Finished` and whose recording process is no longer alive | The session layer appends `Finished { stop: failed, reason: "process exited" }` [REQ-2234] and doesn't report the session as running [REQ-2235] |
| A workspace holds a v0.2.x session store at `.meowg1k/.data/project.db` | The binary neither reads nor migrates it [REQ-2263] |
| A fork names a sequence that doesn't exist, or one inside a superseded range | Forking fails with an error naming the valid range [REQ-2243] |
