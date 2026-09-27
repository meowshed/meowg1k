---
id: ADR-0425
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2823]
supersedes: []
---

# 0425. The plain renderer writes a step's text once and drops deltas

## Decision

The plain renderer drops text deltas and writes a step's text once (from
https://github.com/retran/meowg1k/pull/124, high).

Once this is accepted, the plain renderer ignores `TextDelta` and
`ThinkingDelta` and writes a step's text when it receives a logged `Assistant`
event (from crates/meow-ui/src/plain.rs:69-73 and
crates/meow-ui/src/plain.rs:136-138, high). In a live run that text never
arrives: `Relay` writes the `Assistant` event to the session log and doesn't
send it to the renderer, so the plain renderer prints no model text at all
(from crates/meow-star/src/run.rs:639-647 and
crates/meow-star/src/run.rs:694-707, medium).

## Why

A pipe that received a line per token would be unreadable, and `R-TUI-021` asks
for the same information, not the same granularity (from
https://github.com/retran/meowg1k/pull/124, high). The pull request invites
disagreement (from https://github.com/retran/meowg1k/pull/124, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Write each text delta as it arrives | The text appears as the model produces it, as it does in the terminal (reasoned from crates/meow-ui/src/tty.rs:274-277, low) | A pipe receiving a line per token is unreadable (from https://github.com/retran/meowg1k/pull/124, high) |
| Buffer the deltas in the renderer and write the text at the next step or tool call, as the terminal renderer does | Works from the live stream alone, so the text reaches a pipe without a logged `Assistant` event (from crates/meow-ui/src/tty.rs:263-277, medium) | The history doesn't say why the plain renderer takes the text from the logged event in place of buffering; the renderer's comment says only that the whole text is written once per step (from crates/meow-ui/src/plain.rs:69-72, low) |

A consumer who wants every delta has `--format json`, which serialises each
event it receives (from crates/meow-ui/src/json.rs:55-70, high). Doing nothing
isn't an option here, because before this change there was no plain renderer
(from https://github.com/retran/meowg1k/pull/124, high).

## What it costs

A reader of a pipe sees no text until the step ends, where the terminal shows
it as it arrives (reasoned from crates/meow-ui/src/plain.rs:73 and
crates/meow-ui/src/tty.rs:274-277, low). As wired today the cost is larger: the
plain output of a live run carries no model text, because the event it waits
for never reaches it (from crates/meow-star/src/run.rs:639-647, medium).

## What would reverse it

- A consumer of the plain output that needs the text as it arrives, such as a
  tool that shows progress from `tee`, reported against the renderer (reasoned
  from docs/design/0.3.0-tui.md:180, where plain is meant to be stable under
  `tee`, low).

## Consequences

- The plain renderer's output is one line per tool call, per note and per
  total, as the design's sample shows, and it contains no model prose (from
  docs/design/0.3.0-tui.md:264-275, high).
- `Relay` has to deliver a step's whole text as a logged `Assistant` event for
  the decision to hold, which it doesn't do today (from
  crates/meow-star/src/run.rs:639-647, medium).

## How I will know it was realised

1. `the_plain_renderer_carries_the_same_information` in
   crates/meow-ui/tests/renderers.rs checks the steps, the tool calls with their
   policy decisions and the totals (from
   crates/meow-ui/tests/renderers.rs:185-205, high).
2. That test's recording carries two text deltas and no `Assistant` event, and
   it doesn't assert that the text appears, so no test yet shows a step's text
   written once; a test that runs a handler through `Relay` into the plain
   renderer and finds the model's text on one line would (from
   crates/meow-ui/tests/renderers.rs:17-72, high).

## What this does not settle

- Whether REQ-2823's "every step" includes the model's text; the terminal
  renderer shows it and the plain renderer doesn't in a live run (from
  crates/meow-ui/src/tty.rs:263-277 and crates/meow-star/src/run.rs:639-647,
  medium).
- What happens to thinking deltas, which the terminal shows in the live region
  and never commits (from crates/meow-ui/src/tty.rs:279-282, high).
