---
id: ADR-0411
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2018]
supersedes: []
---

# 0411. A call no rule matches is denied

## Decision

Nothing matching means deny, so an empty policy fails closed (from
https://github.com/retran/meowg1k/pull/119, high). `Policy::evaluate` returns
`Deny` with the source `NoMatch` and no rule named when no rule matches (from
crates/meow-policy/src/policy.rs:40-45 and
crates/meow-policy/tests/spec.rs:237-245, high).

Once this is accepted, a workspace that declares a policy denies every tool it
doesn't grant. What still doesn't hold: a workspace that declares no policy at
all gets none, and an agent with no policy runs every tool call unchecked. The
engine evaluates only when `AgentSpec::policy` is `Some`, and the runtime sets
it to `None` when neither the agent nor the workspace declares one (from
crates/meow-agent/src/engine.rs:358, crates/meow-agent/src/spec.rs:63-66 and
crates/meow-star/src/run.rs:362-365, high). `meow policy show` tells that user
"no policy is declared, so every tool needs a grant", which the engine doesn't
do (from crates/meow-cli/src/wire.rs:537, high).

## Why

Failing closed makes forgetting to grant something a refusal rather than a hole,
and that is what makes the rest of the policy mean anything (from
https://github.com/retran/meowg1k/pull/119, high). A tool an agent was never
granted stays unreachable however convincingly it is asked for (from
crates/meow-policy/src/policy.rs:43-44, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: no policy layer, as in v0.2.x, where `shell_exec` is an ordinary tool | An agent runs every tool with no rule to write and no prompt to answer (from docs/spec/policy.md, Changes from v0.2.x, high) | An agent that is talked into running a command runs it (from docs/spec/policy.md, Changes from v0.2.x, high) |
| Allow a call no rule matches | A new tool works without editing the policy, so a policy lists only what it forbids (reasoned from crates/meow-policy/src/policy.rs:40-45, low) | Forgetting to grant something becomes a hole rather than a refusal (from https://github.com/retran/meowg1k/pull/119, high) |
| Deny a call no rule matches (chosen) | Forgetting a grant is a refusal the user sees (from https://github.com/retran/meowg1k/pull/119, high) | It won |

## What it costs

A workspace has to grant every tool its agents use, and the one in this
repository ends its rules with a catch-all `ask` so an unlisted tool prompts
instead of failing (from .meow/meow.star:63-69, high). Its comment names the
cost: an empty policy would deny everything, which is the safe default and not
a useful one (from .meow/meow.star:64-65, high).

## What would reverse it

- `R-POLICY-011` is amended to let an unmatched call through, which removes
  the requirement this decision implements (reasoned from
  docs/spec/policy.md [R-POLICY-011], low).

## Consequences

- Every decision names its rule or records that none matched, so a denial by
  default reads as "no rule matched" in the transcript (from
  crates/meow-policy/src/policy.rs:48-55, high).
- A workspace needs a policy before its agents can use a tool; this
  repository's `.meow/meow.star` declares one (from .meow/meow.star:66-69,
  high).

## How I will know it was realised

1. `a_call_that_matches_no_rule_is_denied` in crates/meow-policy/tests/spec.rs
   passes: an empty `Policy` denies a call with the source `NoMatch` and no rule
   (from crates/meow-policy/tests/spec.rs:237-245, high).
2. An agent run in a workspace that declares no policy has a tool call refused;
   no test checks this, and today the call runs (from
   crates/meow-star/src/run.rs:362-365, high).

## What this does not settle

- Whether an undeclared policy counts as an empty one. The engine treats
  `None` as "permit everything", which its comment says is only right for a
  test, while the runtime passes `None` in production when nothing is declared
  (from crates/meow-agent/src/spec.rs:63-66 and
  crates/meow-star/src/run.rs:362-365, high).
- What a handler calling a capability module directly may do: policy governs
  what a model decided, not what a script did (from docs/spec/policy.md,
  Decisions, high).
