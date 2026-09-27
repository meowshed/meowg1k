---
id: ADR-2004
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2018, REQ-2037, REQ-2038]
supersedes: []
---

# 2004. Every tool call a model makes passes a policy layer in the runtime

## Decision

The engine evaluates a policy before a tool executes and doesn't execute a tool
whose decision is `deny` (from docs/spec/policy.md [R-POLICY-040], high), and a
call that matches no rule is denied (from docs/spec/policy.md [R-POLICY-011],
high). The policy layer lives in the runtime, outside any Starlark library (from
docs/spec/policy.md, high).

Once this is accepted, `meow-policy` is a crate of its own that matches a call
and returns a decision, and the engine's `invoke` asks it before checking the
arguments or running the tool (from crates/meow-agent/src/engine.rs:357-399,
high). What doesn't hold yet: the engine skips the policy when the agent has
none, and the runtime gives an agent none when neither the agent nor the
workspace declares one, so in that case every tool runs (from
crates/meow-agent/src/engine.rs:358, crates/meow-star/src/run.rs:362-365 and
crates/meow-agent/src/spec.rs:62-67, high).

## Why

v0.2.x had no policy layer and `shell_exec` was an ordinary tool, so an agent
that was talked into running a command ran it (from docs/spec/policy.md, high).
The policy layer is the one part of the system a script must not be able to
weaken, and a library can't be trusted by the thing it constrains (from
docs/spec/policy.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no policy layer, as in v0.2.x, with `shell_exec` an ordinary tool | No rules to write and no prompts, so an agent uses every tool it is given (reasoned from docs/design/0.3.0-architecture.md section 2, low) | An agent that is talked into running a command runs it (from docs/spec/policy.md, high), and a tool that shells out is the product's central safety question with no answer (from docs/design/0.3.0-architecture.md section 2, high) |
| A policy written as a Starlark library | Rules would be written and extended in the language the workspace already uses (reasoned from docs/design/0.3.0-starlark-api.md, low) | A library can't be trusted by the thing it constrains (from docs/spec/policy.md, high) |
| Allow a call that no rule matches | An agent works before anyone writes a rule (reasoned from https://github.com/meowshed/meowg1k/pull/119, low) | Forgetting to grant something becomes a hole and not a refusal (from https://github.com/meowshed/meowg1k/pull/119, high) |

## What it costs

A workspace has to write rules before an agent's tools do anything, because an
empty policy denies every call (from crates/meow-agent/src/spec.rs:62-67 and
https://github.com/meowshed/meowg1k/pull/119, high). The runtime has to describe
each call to the policy, resolving its paths and parsing its command and host
before the decision (from crates/meow-star/src/run.rs:515-566, high).

## What would reverse it

- An isolation boundary outside the process, such as a sandbox every tool runs
  in, that made the in-process check redundant would reopen the question
  (reasoned from docs/spec/policy.md, Scope of trust, low).

## Consequences

The whole policy specification is new in v0.3.0, and it is the largest single
addition of the rewrite (from docs/spec/policy.md, high). A denied call tells
the model which tool policy refused, and a run that policy stopped ends with
the stop reason `denied`, apart from a tool failure (from
https://github.com/meowshed/meowg1k/pull/119 and
crates/meow-agent/tests/spec.rs:761-829, high).

## How I will know it was realised

1. `policy_stops_a_tool_before_it_runs_and_says_so` in
   crates/meow-agent/tests/spec.rs passes: the tool never runs and the decision
   is recorded (from crates/meow-agent/tests/spec.rs:761-801, high).
2. `a_call_that_matches_no_rule_is_denied` in crates/meow-policy/tests/spec.rs
   passes: an empty policy returns `deny` with no rule named (from
   crates/meow-policy/tests/spec.rs:237-245, high).
3. A test running an agent in a workspace that declares no policy shows its
   tool calls denied. No such test exists, and the code runs them (from
   crates/meow-agent/src/engine.rs:358, high).

## What this does not settle

- Whether a workspace with no `meow.policy` gets an empty policy that denies
  everything or no policy at all, as the runtime gives it today (from
  crates/meow-star/src/run.rs:362-365, high).
- How an agent's own policy combines with the workspace's: the runtime uses the
  agent's in place of the workspace's, and `Policy::narrowed_by` is called only
  from tests (from crates/meow-star/src/run.rs:362-365 and a search for
  `narrowed_by` across crates, high).
- ADR-0411 records the no-match denial for REQ-2018 on its own (from
  docs/adrs/ADR-0411-a-call-no-rule-matches-is-denied.md, high).
