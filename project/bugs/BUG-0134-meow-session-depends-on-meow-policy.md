---
id: BUG-0134
artifact: bug
status: approved
severity: minor
violates: REQ-3003
found: 2026-09-27
revised: 2026-09-27
issue:
---

# `meow-session` depends on `meow-policy` for one constant

`meow-session` lists `meow-policy` as a dependency to read
`meow_policy::REDACTED` when it exports, which the crate graph forbids.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. `grep -n meow-policy crates/meow-session/Cargo.toml` prints line 15,
   `meow-policy = { path = "../meow-policy" }`.
2. `grep -rn meow_policy crates/meow-session/src` prints
   `crates/meow-session/src/export.rs:230`.

## What the system does

`export.rs:230` replaces a redacted value with
`meow_policy::REDACTED.to_owned()`. That's the crate's only use of
`meow-policy`.

## What it should do, and why

REQ-3003: "`meow-session` MUST NOT depend on a workspace crate other than
`meow-store` and `meow-core`." The placeholder text should live in `meow-core`,
which both crates may depend on.

## Triage

The defect enters in `meow-session`'s manifest. It's minor: no behaviour is
wrong, but it's the kind of edge that BUG-0135's missing check would have
caught.

## Closed by

The check BUG-0135 asks for, failing on this edge before the fix and passing
after the constant moves to `meow-core`.
