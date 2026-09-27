---
id: BUG-0116
artifact: bug
status: approved
severity: critical
violates: REQ-2260
found: 2026-09-27
revised: 2026-09-27
issue: 239
---

# `meow session export` and `show` never redact a sensitive value

Both commands build a `Redaction` with no argument names, so the export
replaces nothing, whatever the workspace's policy marks sensitive.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Record a session as in BUG-0115, with `token` marked sensitive.
2. Run `meow session export <id>` and `meow session export <id> --as json`.
3. Both print `hunter2`. `meow session show <id>` does too.

This follows from the code below.

## What the system does

`meow session export` builds `Redaction { arguments: Vec::new(), ... }` at
`crates/meow-cli/src/wire.rs:712-715`, and `meow session show` passes
`Redaction::default()` at `wire.rs:667`. The session commands don't load the
workspace's policy, so they have no names to pass. The export itself redacts
correctly when given names, `crates/meow-session/tests/fork.rs:340`.

## What it should do, and why

REQ-2260: "Export MUST redact every value the policy marked sensitive under
[R-POLICY-060], in both formats." REQ-2042 asks for redaction "in every
export". The commands should build `Redaction::arguments` from the workspace
policy's sensitive names.

## Triage

The defect enters in `meow-cli`'s session commands, which don't pass the policy
to the export. It's critical for the same reason as BUG-0115: an export is made
to be shared, so the secret leaves the machine. Fixing BUG-0115 alone keeps the
value out of new logs; this fix covers logs already written.

## Closed by

A test in `crates/meow-cli/tests/`, named for example
`export_redacts_what_the_policy_marks_sensitive`, that exports a session with a
sensitive argument in both formats and expects `[REDACTED]` in place of the
value.
