---
id: BUG-0017
artifact: bug
status: approved
severity: critical
violates: REQ-2018
found: 2026-09-27
revised: 2026-09-27
issue: 190
---

# With no declared policy, every tool call runs unchecked

A workspace that declares no `meow.policy` builds agent specs with `policy:
None`, and the engine evaluates policy only when a policy is present. Every tool
call then runs with no decision, no prompt and no `Policy` event, while
`meow trust` and `meow policy show` tell the user the opposite.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Create a workspace whose `.meow/meow.star` declares a command and no
   `meow.policy`.
2. Run `meow trust` with `HOME` pointed at a scratch directory. It prints
   `policy none, so every tool call is asked`.
3. Run `meow policy show`. It prints `no policy is declared, so every tool needs
   a grant`.
4. Run an agent with one tool against a recorded model reply that calls the
   tool. The tool runs, no approval is asked, and the session log holds no
   `Policy` event.

Steps 1 to 3 were run against `target/debug/meow`; step 4 was traced through
the code below and not run.

## What the system does

`crates/meow-star/src/run.rs:362-365` sets `policy` to the agent's own policy or
else the workspace's, which is `None` when neither is declared.
`crates/meow-agent/src/engine.rs:358` evaluates only inside `if let
(Some(policy), Some(describe))`, so with `None` the call goes straight to
`tool.call` at `crates/meow-agent/src/engine.rs:406-419`. The messages that say
otherwise are at `crates/meow-cli/src/trust.rs:64` and
`crates/meow-cli/src/wire.rs:537`.

## What it should do, and why

REQ-2018: "A call that matches no rule MUST be denied." With no policy, no rule
matches any call. REQ-2037 asks the engine to evaluate the policy before the
tool executes, and REQ-2250 asks for a `Policy` event per decision; both are
skipped on this path.

## Triage

The defect enters where `meow-star` builds the `AgentSpec` and meets the
engine's optional policy. It is critical: the vision's first goal treats
`.meow/` as code with tool access, and here the default-deny boundary is absent
in exactly the workspace that declared nothing, while the trust prompt tells the
user every call will be asked. The fix is to evaluate an empty `Policy` when none
is declared, which denies or asks by REQ-2018 and writes the `Policy` event.

This record also covers two symptoms of the same cause, listed in the defect
list as separate items: `meow policy show` claiming a grant is needed, and the
missing `Policy` event when no policy is declared. Both are correct once the
empty policy is evaluated.

## Closed by

A test in `crates/meow-star/tests/`, named for example
`an_undeclared_policy_denies_every_tool_call`, that runs an agent with one tool
in a workspace with no policy and expects the call refused and a `Policy` event
with decision `deny`.
