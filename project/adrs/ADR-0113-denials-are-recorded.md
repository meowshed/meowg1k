---
id: ADR-0113
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2250, REQ-2251]
supersedes: []
---

# 0113. Every tool call records its policy decision before it runs, denials included

## Decision

Every tool call produces a `Policy` event before it runs, including the allowed
calls and the denied calls that never ran, and `source` tells a rule match from
an interactive approval from a session-scoped `always` (from
docs/design/0.3.0-sessions.md section 10, high).

## Why

v0.2.x had no permission record, so there was nothing to audit (from
docs/design/0.3.0-sessions.md section 2, high). An agent that was talked into
trying something is recorded trying it, and without a log of denials a policy
that works can't be told from a policy that was never tested (from
docs/design/0.3.0-sessions.md section 10, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no permission record, as in v0.2.x | No event to write per call and no schema for decisions (reasoned from docs/design/0.3.0-sessions.md section 2, low) | Nothing to audit (from docs/design/0.3.0-sessions.md section 2, high) |
| Record only the calls that ran | A smaller log, since a denied call leaves only the model's request behind (reasoned from docs/design/0.3.0-sessions.md section 10, low) | A policy that works can't be told from one that was never tested (from docs/design/0.3.0-sessions.md section 10, high) |
| Record the decision after the tool runs | The event could carry the result beside the decision (reasoned from crates/meow-agent/src/engine.rs:357-363, low) | A call that crashes or is cancelled mid-run would leave no decision behind, and REQ-2037 puts the decision before the tool runs, never after (from crates/meow-agent/src/engine.rs:357, high) |

## What it costs

One extra event per tool call, written before the tool runs (from
crates/meow-star/src/run.rs:742-751, high). A record of what was attempted also
holds what an agent was talked into trying, so a session export has to redact
the values the policy marks sensitive (from docs/design/0.3.0-sessions.md
section 9, high).

## What would reverse it

- A requirement to keep a denied call out of the log, for example because its
  arguments are secret, which redaction on export doesn't cover (reasoned from
  crates/meow-session/src/export.rs:12-27, low).

## Consequences

A markdown export lists every tool call with its decision and the rule behind
it (from crates/meow-session/src/export.rs:69-72, high). The design's
`meow session show <id> --tools` isn't built: `meow session show` prints the
markdown export and takes no `--tools` flag (from
crates/meow-cli/src/wire.rs:665-674, high).

## How I will know it was realised

1. `every_policy_decision_is_recorded_including_the_denials` in
   crates/meow-session/tests/spec.rs passes: `allow`, `ask` and `deny` all
   reach the log (from crates/meow-session/tests/spec.rs:531, high).
2. A run whose policy denies a call and one that answers an `ask`
   interactively write `Policy` events whose `rule` names the matching rule and
   whose `source` differs. No test checks this today, and the code fails it (see
   the last item below).

## What this does not settle

- The `source` and `rule` fields are not filled from the decision today. The
  engine's `AgentEvent::Policy` carries only the call and the decision, and the
  runtime writes `rule: None` and `source: "rule"` for every call, so an
  interactive answer and a session grant read as rule matches (from
  crates/meow-agent/src/event.rs:54-60 and crates/meow-star/src/run.rs:742-751,
  high).
- A call with no policy declared for the agent or the workspace writes no
  `Policy` event, because the engine emits one only when a policy is set (from
  crates/meow-agent/src/engine.rs:358 and crates/meow-star/src/run.rs:362-365,
  high).
