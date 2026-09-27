---
id: BUG-0018
artifact: bug
status: approved
severity: major
violates: REQ-2033
found: 2026-09-27
revised: 2026-09-27
issue: 191
---

# An agent's policy replaces the workspace's, so it can allow what the workspace denies

When an agent declares its own policy, the spec takes it in place of the
workspace policy. `Policy::narrowed_by`, which combines the two and keeps the
stricter answer, is called only from tests.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare a workspace policy that denies tool `shell`.
2. Declare an agent whose own policy allows `shell`, and give it `shell` as a
   tool.
3. Run the agent against a recorded reply that calls `shell`.

The call runs. It should be denied.

`grep -rn narrowed_by crates` finds the definition at
`crates/meow-policy/src/policy.rs:344` and callers only under
`crates/meow-policy/tests/`.

## What the system does

`crates/meow-star/src/run.rs:362-365` is
`declared.policy.clone().or_else(|| self.registry.policy().cloned())`: the
agent's policy wins outright, and the workspace policy is read only when the
agent has none.

## What it should do, and why

REQ-2033: "An agent policy MUST NOT allow a call the workspace policy denies."
REQ-2031 and REQ-2032 describe the narrowing, and `narrowed_by` implements it.

## Triage

The defect enters in `meow-star`'s `agent_spec`, which picks one policy where it
should combine two. It is major because the requirement's main behaviour doesn't
happen. The agent and the workspace policy come from the same trusted `.meow/`,
which is why this is not rated critical, but a package can ship an agent, and
then the workspace's boundary is whatever the package says. The fix is to hand
the engine `workspace.narrowed_by(agent)` when both exist.

## Closed by

A test in `crates/meow-star/tests/`, named for example
`an_agent_policy_cannot_allow_what_the_workspace_denies`, that runs the steps
above and expects the call denied.
