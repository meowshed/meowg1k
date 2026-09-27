---
id: BUG-0127
artifact: bug
status: approved
severity: minor
violates: REQ-1213
found: 2026-09-27
revised: 2026-09-27
issue: 250
---

# A `copilot` provider declared under another name can't log in by device code

`meow auth login` picks the device flow when the provider's name is `copilot`,
not when its kind is. A workspace that declares
`meow.provider(name = "gh", kind = "copilot")` gets a key prompt from
`meow auth login gh`.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Declare `meow.provider(name = "gh", kind = "copilot")`.
2. Run `meow auth login gh` in a terminal.
3. It asks for an API key and shows no device code.

This follows from the code below; ADR-0461 reasons the same.

## What the system does

`oauth_kind` at `crates/meow-cli/src/wire.rs:330-332` compares the name with
`"copilot"`. The comment at `wire.rs:93-96` says the kind "comes from the
workspace when there is one", which the code doesn't do.

## What it should do, and why

REQ-1213: "A provider kind that authenticates by OAuth rather than an API key
MUST obtain its credential through a device-code flow". The kind should decide,
read from the workspace when one loads, with the name as the fallback when none
does.

## Triage

The defect enters in `meow auth login`, by ADR-0461's choice. It's minor:
naming the provider `copilot` works around it. The comment at `wire.rs:93`
should be corrected in the same change.

## Closed by

A test in `crates/meow-cli/tests/auth.rs`, named for example
`a_copilot_kind_under_another_name_uses_the_device_flow`, that declares the
provider above and expects the device flow's first request.
