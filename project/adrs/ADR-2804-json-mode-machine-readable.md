---
id: ADR-2804
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2801, REQ-2824, REQ-2828, REQ-2829]
supersedes: []
---

# 2804. `--format json` gives every run a machine-readable mode

## Decision

`--format json` selects a JSON renderer whose stdout carries only the event
stream, and diagnostics go to stderr (from docs/spec/tui.md [R-TUI-002]
[R-TUI-033], high). The source's account of the v0.2.x gap doesn't cite these
requirements; I mapped them, and the code confirms the mapping:
`choose` returns the JSON renderer whenever `--format json` is given, and the
JSON renderer writes only events (from crates/meow-ui/src/lib.rs:100-116 and
crates/meow-ui/src/json.rs:12-21, high).

Once this holds, `meow review --format json | jq` works from an interactive
shell as well as from a pipe (from crates/meow-ui/src/lib.rs:100-116, high).
A diagnostic raised mid-run under `--format json` travels the same way as one
raised while loading, but only the loading path is tested (from
https://github.com/meowshed/meowg1k/pull/151, high).

## Why

v0.2.x has no machine-readable mode, so `meow review \| jq` is impossible (from
docs/spec/tui.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no machine-readable mode, as in v0.2.x | No event schema or version number has to be kept stable for outside consumers (reasoned from docs/spec/tui.md [R-TUI-031] [R-TUI-032], low) | `meow review \| jq` is impossible, and the tool can't be composed with anything (from docs/spec/tui.md and docs/design/0.3.0-tui.md section 3, high) |
| Parse the plain renderer's output | The plain transcript is already line-oriented and greppable, so no second format is needed (from docs/design/0.3.0-tui.md section 5.2, medium) | The design names the JSON renderer, not the plain one, as what a script parses: every event the terminal can draw is an event a script can parse (from docs/design/0.3.0-tui.md section 4, medium) |

No further option appears in the design documents or the pull request history
(from docs/design/0.3.0-tui.md sections 3 and 4, high).

## What it costs

The event schema carries a version, and the live stream and the export must
share it, so changing an event kind is a change to a public format (from
docs/spec/tui.md [R-TUI-031] [R-TUI-032], high). Every diagnostic in
`meow-cli` has to go to stderr, never stdout, which the JSON renderer can't
enforce on its own (from crates/meow-ui/src/json.rs:20-21, high).

## What would reverse it

The live stream and the export can no longer share one schema, as [REQ-2826]
requires, without breaking consumers of one of them; a consumer that needs a
format other than JSON would add a renderer, not remove this one (reasoned
from docs/spec/tui.md [R-TUI-032] and crates/meow-ui/src/lib.rs:76-85, low).

## Consequences

`--format json` wins whatever the terminal is, because a program asking for
JSON gets JSON even from an interactive shell (from
crates/meow-ui/src/lib.rs:100-116, high). The stream is JSON Lines, which
[ADR-2801] records (from docs/spec/tui.md [R-TUI-030], high).

## How I will know it was realised

1. `json_wins_whatever_the_terminal_is` in crates/meow-ui/tests/renderers.rs
   passes.
2. `format_json_produces_one_object_per_line` in
   crates/meow-cli/tests/surface.rs passes: every stdout line of
   `meow --format json greet world` parses as an event.
3. `json_keeps_diagnostics_off_stdout` in the same file passes: a workspace
   that won't load complains on stderr while stdout stays parseable.

## What this does not settle

- Whether a diagnostic raised mid-run under `--format json` reaches stderr;
  no test covers it (from https://github.com/meowshed/meowg1k/pull/151, high).
- `RunStart.model` is carried but left empty, because the engine reports the
  agent and not the model it resolved (from
  https://github.com/meowshed/meowg1k/pull/124 and
  crates/meow-star/src/run.rs:685, high).
