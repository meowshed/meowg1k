---
id: BUG-0019
artifact: bug
status: approved
severity: minor
violates: REQ-2029
found: 2026-09-27
revised: 2026-09-27
issue: 192
---

# An approval prompt's timeout can't be configured

`meow_policy::Timeout` exists and nothing constructs it outside `meow-policy`.
No declaration, flag or setting reaches it, so a prompt waits for ever and the
expiry that REQ-2030 turns into a denial can't happen.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run `grep -rln Timeout crates`. Outside `crates/meow-policy/src/`, no file
   names it.
2. Look for a timeout argument on `meow.policy` in
   `crates/meow-star/src/declare.rs`. It takes none.

## What the system does

`crates/meow-policy/src/prompt.rs:20-22` defines `Timeout` and
`crates/meow-policy/src/lib.rs:28` exports it. Nothing in `meow-star`,
`meow-agent` or `meow-cli` uses it.

## What it should do, and why

REQ-2029: "A timeout on an approval prompt MAY be configured." REQ-2030: "When a
timeout is configured and expires, the approval prompt MUST resolve to `deny`."
The MAY grants the user the option, so the product has to offer a way to set it.

## Triage

The defect enters between `meow-star`'s declarations and the approver
`meow-cli` builds. It is minor: an attended prompt still works, and an
unattended run already turns `ask` into `deny` by REQ-2022. If the MAY is read as
optional for the product, REQ-2030 holds vacuously and the requirement should
say so instead; the record takes the reading that the option is the user's.

## Closed by

A test in `crates/meow-star/tests/` or `crates/meow-cli/tests/`, named for
example `an_expired_approval_prompt_denies`, that configures a short timeout,
answers nothing, and expects the call denied.
