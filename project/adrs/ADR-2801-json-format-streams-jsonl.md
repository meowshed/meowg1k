---
id: ADR-2801
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2824]
supersedes: []
---

# 2801. `--format json` streams one JSON object per line

## Decision

`--format json` emits the event stream as JSON Lines (JSONL), one JSON object
per line (from docs/spec/tui.md [R-TUI-030], high).

Once this holds, a consumer such as `jq -c` reads each event as the run
produces it, because the JSON renderer writes one line per event as it arrives
(from crates/meow-ui/src/json.rs:49-70, high). A consumer that wants one
document has to collect the lines itself (from docs/spec/tui.md, Decisions,
high).

## Why

A consumer can react while the run is going, and anyone who wants one document
is one `jq -s` away. A buffered document can't be turned back into a stream that
arrives while the run is happening (from docs/spec/tui.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| One buffered JSON document | The output is a single JSON document, which a consumer parses in one call (from the open question removed in commit e39cd2f, docs/spec/tui.md, high) | It can't be turned back into a stream that arrives while the run is happening (from docs/spec/tui.md, high) |

The open question that commit e39cd2f settled weighed only streaming against
buffering, and doing nothing wasn't an option, because [REQ-2801] already
required a JSON mode (from docs/spec/tui.md, Open questions before commit
e39cd2f, high).

## What it costs

A stream has no enclosing object to carry its version, so the version travels as
a `Schema` event on the first line, which [REQ-2825] requires and the renderer
emits before anything else (from crates/meow-ui/src/json.rs:56-70, high). A
consumer that wants one document pays a `jq -s` (from docs/spec/tui.md, high).
A run killed mid-stream leaves a valid prefix with no closing event, so a
consumer has to treat a missing `RunEnd` as a run that didn't finish (reasoned
from crates/meow-core/src/view.rs:135, low).

## What would reverse it

A supported consumer that can read only one JSON document, and can't run
`jq -s` or its equivalent, would reverse it (reasoned from docs/spec/tui.md,
Decisions, low).

## Consequences

A consumer that wants one document runs `jq -s` over the stream (from
docs/spec/tui.md, high). `meow session export --format json` writes the same
one-object-per-line form, since it serialises each event through the same
`ViewEvent` type (from crates/meow-session/src/export.rs:48-60, high).

## How I will know it was realised

1. `the_json_renderer_announces_its_schema_first` in
   crates/meow-ui/tests/renderers.rs passes: every line parses as an object
   with a `type`, and the first is `Schema`.
2. `format_json_produces_one_object_per_line` in
   crates/meow-cli/tests/surface.rs passes against the built binary.

Neither test checks that a line reaches the consumer before the run ends; that
rests on the renderer writing each event with `writeln!` to a line-buffered
stdout (reasoned from crates/meow-ui/src/json.rs:49-52, low).

## What this does not settle

- What each event kind carries and how the schema version changes, which
  [REQ-2826] and [REQ-2827] govern (from docs/spec/tui.md [R-TUI-032], high).
- Which kinds may appear only in the live stream, which [REQ-2830] and
  [REQ-2831] govern (from docs/spec/tui.md [R-TUI-034], high).
