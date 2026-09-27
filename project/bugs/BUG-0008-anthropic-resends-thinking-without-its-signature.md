---
id: BUG-0008
artifact: bug
status: approved
severity: minor
violates: REQ-1623
found: 2026-09-27
revised: 2026-09-27
issue: 181
---

# Anthropic sends thinking back without its signature

The Anthropic provider reads a `thinking` block's text and drops its
`signature`, then sends the block back as `{"type": "thinking", "thinking":
...}`. Anthropic's Messages API requires the signature on a thinking block
returned in a later turn, so the request that continues a tool call would be
refused.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run `grep -rn signature crates/meow-llm/src`. It finds nothing.
2. Read the test at
   `crates/meow-llm/tests/spec.rs:315-341`: the recorded response carries a
   thinking block without a signature, and the test checks only that a block of
   type `thinking` goes back.

## What the system does

`crates/meow-llm/src/anthropic.rs:203-205` keeps only `block["thinking"]`, and
`crates/meow-llm/src/anthropic.rs:101-106` sends back a block with the text
alone. `Message::thinking` is an `Option<String>`, so the signature has nowhere
to go.

## What it should do, and why

REQ-1623: "Thinking content MUST be preserved on the assistant message it
belongs to, because a provider can require it to be sent back on a later turn
that continues a tool call." A block sent back without the signature Anthropic
issued isn't the block it gave, and the requirement's reason fails. ADR-1602
records the decision.

## Triage

The defect enters in `meow-llm`'s message type and the Anthropic provider. It is
minor and dormant: no request sets Anthropic's `thinking` parameter, so no
thinking block arrives today. It fails on the first run that enables extended
thinking with tools.

## Closed by

A test in `crates/meow-llm/tests/spec.rs`, named for example
`a_thinking_block_goes_back_with_its_signature`, that records a thinking block
carrying a signature and expects the next request to send both.
