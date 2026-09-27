---
id: ADR-2003
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2028, REQ-2029, REQ-2030]
supersedes: []
---

# 2003. An approval prompt waits indefinitely by default

## Decision

An approval prompt waits indefinitely by default; a timeout can be configured,
and when a configured timeout expires the prompt resolves to `deny` (from
docs/spec/policy.md [R-POLICY-024], high).

Once this is accepted, `Timeout::default()` holds no duration, and an expired
timeout resolves to `deny` and never to `allow` (from
crates/meow-policy/src/prompt.rs:12-30, high). The terminal approver blocks on
a line from standard input with no deadline (from
crates/meow-cli/src/ask.rs:88-103, high). No declaration or flag sets a
timeout yet, and the `Approver` trait leaves it to the implementation (from
crates/meow-agent/src/spec.rs:21-33 and a search for `Timeout` across
crates/meow-star/src and crates/meow-cli/src, which finds no use, high).

## Why

A prompt that expires while you're reading the command it asks about turns a
security decision into a reflex (from docs/spec/policy.md, high). The timeout
stays configurable for an unattended terminal that is still a terminal (from
docs/spec/policy.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| A prompt that expires by default | A run left at a prompt ends on its own, with the call denied (reasoned from crates/meow-policy/src/prompt.rs:22-29, low) | A prompt that expires while you're reading the command turns a security decision into a reflex (from docs/spec/policy.md, high) |
| A prompt that waits with no timeout at all | Nothing to configure, and the Approver trait needs no deadline (reasoned from crates/meow-agent/src/spec.rs:21-33, low) | An unattended terminal that is still a terminal needs a way to end the wait (from docs/spec/policy.md, high) |
| An expired prompt resolving to `allow` | An unattended run makes progress (reasoned from docs/design/0.3.0-tui.md section 7, low) | Degrading to `allow` would make the policy decorative (from docs/design/0.3.0-tui.md section 7, which says it of a non-interactive `ask`, medium) |

## What it costs

A run left at a prompt holds its session and the process until a person
answers, because nothing ends the wait by default (reasoned from
crates/meow-cli/src/ask.rs:88-103, low).

## What would reverse it

- A report that runs left at a prompt, on terminals nobody watches, are common
  enough that a default timeout is the safer default would reopen the question
  (reasoned from docs/spec/policy.md, Decisions, low).

## Consequences

A person reading a long command isn't rushed into an answer (from
docs/spec/policy.md, Decisions, high). Every outcome, a timeout among them, is
recorded in the session log as a policy event (from docs/design/0.3.0-tui.md
section 7, high).

## How I will know it was realised

1. `a_prompt_waits_unless_a_timeout_was_configured` in
   crates/meow-policy/tests/spec.rs passes: the default has no duration and an
   expiry resolves to `deny` (from crates/meow-policy/tests/spec.rs:520-536,
   high).

## What this does not settle

- Where a person configures the timeout: no declaration, flag or setting
  reaches `Timeout` today (from a search for `Timeout` across
  crates/meow-star/src and crates/meow-cli/src, high).
- How long a configured timeout may be, and whether the prompt shows it
  counting down (from docs/spec/policy.md [R-POLICY-024], which gives no
  value, medium).
