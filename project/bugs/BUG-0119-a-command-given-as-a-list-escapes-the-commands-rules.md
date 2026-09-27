---
id: BUG-0119
artifact: bug
status: approved
severity: major
violates: REQ-2008
found: 2026-09-27
revised: 2026-09-27
issue: 242
---

# A `command` argument given as a list escapes every `commands` rule

The policy reads a tool call's `command` argument only when it's a string, and
ADR-0445 makes a command a list of words, so a call whose `command` is a list
reaches the policy with no command at all and no `commands` rule can match it.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare a tool `sh` whose `command` argument is a list of strings and whose
   handler passes it to `shell.run`.
2. Declare `meow.policy(rules = [{"commands": ["rm *"], "decision": "deny"}, {"tools": ["sh"], "decision": "allow"}])`.
3. Drive an agent with a scripted model that calls `sh` with
   `{"command": ["rm", "-rf", "build"]}`.
4. The call is allowed by the `tools` rule and runs.

This follows from the code below.

## What the system does

`describe_call` in `crates/meow-star/src/run.rs:557-559` sets the command only
from `args.get("command").and_then(Value::as_str)`, so a JSON array leaves the
`Call` without one. `shell.run` and `shell.capture` refuse a string command,
`crates/meow-star/src/capability.rs:316-340`, so a tool that runs a command
takes a list.

## What it should do, and why

REQ-2008: "A `commands` selector MUST match against the full command line as a
single string, using glob semantics." A list of words should be joined into
that string, with each word quoted where it holds a space, and judged.

## Triage

The defect enters in `meow-star`'s `describe_call`. It's major because a deny
rule a person wrote for commands is ignored for the only command shape the
runtime accepts, and the call then falls to broader rules. It isn't critical
because it needs a workspace that allows the tool elsewhere; with no other
rule, the call is still refused.

## Closed by

A test in `crates/meow-star/tests/approval.rs`, named for example
`a_command_list_meets_a_commands_rule`, running the reproduction above and
expecting the call denied.
