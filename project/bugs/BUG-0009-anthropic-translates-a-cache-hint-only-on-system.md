---
id: BUG-0009
artifact: bug
status: approved
severity: minor
violates: REQ-1614
found: 2026-09-27
revised: 2026-09-27
issue: 182
---

# Anthropic translates a cache hint only on a system message

The Anthropic provider turns `cache_hint` into a `cache_control` breakpoint on a
system message and ignores it on a user, assistant or tool message. Nothing in
production sets a cache hint on any message either, so prompt caching never
reaches Anthropic's API.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Build a `Request` whose user message carries `.with_cache_hint()`.
2. Send it through `Anthropic::generate` with a recorded transport, as
   `a_cache_hint_becomes_a_breakpoint_without_changing_the_message` in
   `crates/meow-llm/tests/spec.rs` does for a system message.
3. Read the sent body. The user message has no `cache_control`.

Separately, `grep -rn cache_hint crates` finds no production caller of
`with_cache_hint`.

## What the system does

`crates/meow-llm/src/anthropic.rs:83-91` reads `m.cache_hint` in the
`Role::System` arm only. The `Role::User`, `Role::Assistant` and `Role::Tool`
arms at `crates/meow-llm/src/anthropic.rs:91-120` never read it.
`Message::with_cache_hint` (`crates/meow-llm/src/message.rs:97`) is called only
from tests.

## What it should do, and why

REQ-1614: "A provider whose API has explicit cache breakpoints MUST translate a
message's cache hint into one." The requirement says a message, not a system
message, and Anthropic accepts `cache_control` on any content block. ADR-1601
records the decision.

## Triage

The defect enters in the Anthropic provider's body builder. It is minor: the
answers are right and only the cost is higher, and with no caller setting a hint
the missing translation changes nothing today. Whether the engine should set a
hint is a separate question REQ-1613 leaves open with a MAY.

## Closed by

A test in `crates/meow-llm/tests/spec.rs`, named for example
`a_cache_hint_on_a_user_message_becomes_a_breakpoint`, that expects
`cache_control` on the user message's content block.
