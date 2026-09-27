---
id: BUG-0108
artifact: bug
status: approved
severity: major
violates: REQ-2250
found: 2026-09-27
revised: 2026-09-27
issue: 231
---

# The `Policy` event omits the rule and records the decision before the answer

Every `Policy` event the engine writes has `rule: None` and `source: "rule"`,
and it's written before an `ask` is answered, so an approval, a denial by the
person or a session grant is logged as `ask` from a rule.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare `meow.policy(rules = [{"tools": ["*"], "decision": "ask"}])` and an
   agent with one tool.
2. Run it with a terminal and approve the call once.
3. Run `meow session export <id> --as json`. The `Policy` event reads
   `decision: "ask"`, `rule: null`, `source: "rule"`.

This follows from the code below.

## What the system does

`AgentEvent::Policy` has only `id` and `decision`,
`crates/meow-agent/src/event.rs:54-60`. The engine sends it with the verdict's
decision before it asks the approver, `crates/meow-agent/src/engine.rs:357-376`.
`meow-star` turns it into the log's event with `rule: None` and
`source: "rule"` fixed, `crates/meow-star/src/run.rs:742-751`.

## What it should do, and why

REQ-2250: "Every tool invocation MUST produce a `Policy` event before the tool
runs, recording the decision, the rule that produced it, and whether the
decision came from a rule, an interactive answer, or a session grant." The
event should carry the matched rule, the final decision and its source.

## Triage

The defect enters in `meow-agent`'s event type, which has no field for the rule
or the source, and in the order the engine sends it. It's major because two of
the requirement's three facts are never recorded and the third is wrong for
every answered prompt, so the audit log can't show who allowed a call.

## Closed by

A test in `crates/meow-star/tests/approval.rs`, named for example
`an_approved_call_records_the_rule_and_the_answer`, that approves one call and
expects a `Policy` event with the rule text, decision `allow` and source
`answer`.
