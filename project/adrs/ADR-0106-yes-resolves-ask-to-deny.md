---
id: ADR-0106
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2022, REQ-2848, REQ-2849]
supersedes: []
---

# 0106. Under `--yes` or with no terminal, an `ask` decision resolves to deny

## Decision

Under `--yes`, `Ask` degrades to `Deny` and never to `Allow` (from
docs/design/0.3.0-architecture.md section 5.3, high). With `--yes` or no TTY,
`ask` resolves to `deny` and the decision is recorded (from
docs/design/0.3.0-tui.md section 7, high). The binary checks `--yes` before the
terminal, so `--yes` in an interactive shell still counts as unattended (from
crates/meow-cli/src/ask.rs:32-44, high).

## Why

An unattended run in CI can't silently escalate (from
docs/design/0.3.0-architecture.md section 5.3, high). An unattended run gets
strictly less authority than an attended one, never more (from
docs/design/0.3.0-tui.md section 2.3, high). The source calls this the one
default that would be quietly disastrous to get backwards (from
docs/design/0.3.0-tui.md section 2.3, high). The flag is a statement about how
the run should behave, not about the terminal (from
https://github.com/retran/meowg1k/pull/127, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no permission boundary, as in v0.2.x, so every tool call runs | An unattended run never stops on a question, because there is none (reasoned from docs/design/0.3.0-tui.md section 3, low) | An LLM-driven tool that shells out is the product's central safety question, and v0.2.x had no answer to it (from docs/design/0.3.0-architecture.md section 2, high) |
| `ask` degrades to `allow` when nobody can answer | A CI run can use every tool a rule marks `ask` without editing the policy (reasoned from docs/design/0.3.0-tui.md section 2.3, low) | An unattended run could silently escalate, and it would make the policy decorative (from docs/design/0.3.0-tui.md sections 2.3 and 7, high) |
| Wait for an answer that can't come | Never decides on the user's behalf (reasoned from docs/design/0.3.0-tui.md section 1.1, low) | Both interactive moments have to degrade to a refusal rather than a hang when there is no terminal (from docs/design/0.3.0-tui.md section 1.1, high) |

## What it costs

A run under `--yes` or in CI can't use a tool whose rule says `ask`; the author
has to move that call to an `allow` rule to automate it (reasoned from
crates/meow-star/tests/approval.rs:249-272, low). An agent that needed the
refused tool stops, and the binary reports a denial with its own exit code, 5
(from docs/design/0.3.0-tui.md section 2.4, high).

## What would reverse it

- A way to grant consent ahead of an unattended run that the session log records
  as a person's decision, which would give `ask` somebody to answer it (reasoned
  from docs/design/0.3.0-tui.md section 7, low). While consent can only come
  from a prompt, nothing else can answer `ask`.

## Consequences

Every outcome, a timeout included, is appended to the session log as a policy
decision event, so `meow session show` reconstructs what the agent was permitted
to do (from docs/design/0.3.0-tui.md section 7, high). The model gets the
denial as a tool result and can carry on without the tool (from
crates/meow-star/tests/approval.rs:249-272, high). `ctx.ask` follows the same
rule and fails in place of blocking (from docs/design/0.3.0-tui.md section 6,
high).

## How I will know it was realised

1. `an_ask_with_nobody_to_answer_becomes_deny_not_allow` in
   crates/meow-policy/tests/spec.rs passes (from
   crates/meow-policy/tests/spec.rs:301-320, high).
2. `an_ask_with_nobody_to_ask_is_a_denial` in crates/meow-star/tests/approval.rs
   passes: the tool doesn't run and the transcript shows `policy: ask` (from
   crates/meow-star/tests/approval.rs:249-272, high).
3. `yes_refuses_rather_than_approving` in crates/meow-cli/tests/surface.rs and
   `yes_is_unattended_and_is_not_consent` in crates/meow-cli/src/ask.rs pass
   (from crates/meow-cli/tests/surface.rs:346-369 and
   crates/meow-cli/src/ask.rs:336-347, high).

## What this does not settle

- What a prompt does when a terminal is present but nobody answers: it waits
  indefinitely unless a timeout is configured, and an expired timeout resolves
  to `deny` (from docs/spec/policy.md [R-POLICY-024], high).
- How the denial is recorded and what stop reason the run gets, which are
  separate decisions (from docs/design/0.3.0-tui.md section 7, high).
