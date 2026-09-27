---
id: BUG-0049
artifact: bug
status: approved
severity: major
violates: REQ-2823
found: 2026-09-27
revised: 2026-09-27
issue:
---

# The plain renderer prints none of the model's text in a live run

The plain renderer drops text deltas and prints the model's text only from a
logged `Assistant` event. The relay writes that event to the session log without
sending it to the renderer, so with `--plain` or a piped stdout, no step's model
text appears.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run an agent command against a recorded provider whose reply is text, with
   stdout piped to `cat`.
2. Read the output.

It shows the run header, the tool lines and the totals, and no model text. This
was traced through the code and not run.

## What the system does

`crates/meow-ui/src/plain.rs:70-74` drops `TextDelta` and `ThinkingDelta`, and
`crates/meow-ui/src/plain.rs:137-142` prints `Logged(EventKind::Assistant)`.
`Relay::wrote` at `crates/meow-star/src/run.rs:639-647` calls
`self.log.record(...)` and never `self.send(...)`, unlike `Relay::keep` at
`crates/meow-star/src/run.rs:653-656`, which does both.

## What it should do, and why

REQ-2823: "The plain renderer MUST carry the same information as the terminal
renderer: every step, every tool call with its policy decision, and the
totals." A step's text is the step. ADR-0425 records the decision.

## Triage

The defect enters in `meow-star`'s `Relay`, which picks `record` where `keep`
was meant. It is major because the plain renderer is what CI and scripts see,
and there it shows nothing the model said. The fix is for `Relay::wrote` to call
`keep`.

## Closed by

A test in `crates/meow-star/tests/` or `crates/meow-cli/tests/`, named for
example `the_plain_renderer_prints_each_steps_text`, that runs a recorded agent
through the plain renderer and expects the reply's text in its output.
