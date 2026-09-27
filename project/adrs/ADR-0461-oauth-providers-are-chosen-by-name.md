---
id: ADR-0461
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1213]
supersedes: []
---

# 0461. Which provider authenticates by OAuth is decided by its name

## Decision

Which providers use OAuth is decided by name, not by the workspace's declared
kind (from https://github.com/meowshed/meowg1k/pull/155, high).

Once this is accepted, `oauth_kind` answers yes for the name `copilot` in any
case and no for every other name, and `meow auth login copilot` runs the device
flow while every other name prompts for a key (from
crates/meow-cli/src/wire.rs:97 and crates/meow-cli/src/wire.rs:330, high). A
workspace provider of kind `copilot` declared under another name, say `gh`,
can't obtain its grant by the device flow, because `meow auth login gh` prompts
for a key (reasoned from crates/meow-cli/src/wire.rs:97, medium). The comment
above the call says the kind comes from the workspace when there is one, which
the code doesn't do (from crates/meow-cli/src/wire.rs:93, high).

## Why

`meow auth login` needs no workspace, and a name is all it has (from
https://github.com/meowshed/meowg1k/pull/155, high). The kinds that use OAuth are
few and fixed, so a table is honest (from crates/meow-cli/src/wire.rs:325,
high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Decide by the workspace's declared kind | A provider of kind `copilot` gets the device flow whatever it is named (reasoned from crates/meow-cli/src/wire.rs:1431, low) | `meow auth login` runs without a workspace (from https://github.com/meowshed/meowg1k/pull/155, high) |
| A subcommand per OAuth provider, as v0.2.x's `meow auth copilot` | No name can be mistaken for a kind, because the command names the flow (from `git show v0.2.1:cmd/auth.go`, medium) | The specification's shape is one `meow auth login <provider>` for every provider (from docs/spec/auth.md [R-AUTH-013], high) |
| Do nothing: no OAuth path, so every provider takes a key | One login path (reasoned from crates/meow-cli/src/wire.rs:101, low) | REQ-1213 requires a provider kind that authenticates by OAuth to obtain its credential by a device-code flow (from docs/spec/auth.md [R-AUTH-020], high) |

## What it costs

A provider named `copilot` with some other kind takes the OAuth path, a mistake
this makes visible rather than causes (from
https://github.com/meowshed/meowg1k/pull/155, high). A provider of kind `copilot`
under another name can't use the device flow, and the credential lookup is by
provider name, so it has to be named `copilot` to find the grant the flow stores
(from crates/meow-cli/src/wire.rs:1431, high).

## What would reverse it

A second OAuth kind, or a workspace that needs two Copilot providers under two
names, would make a table of names the wrong key (reasoned from
crates/meow-cli/src/wire.rs:330, low). So would `meow auth login` gaining a
workspace to read kinds from.

## Consequences

- `meow auth login copilot` works in an empty directory, because it reads no
  workspace (from crates/meow-cli/src/wire.rs:97, high).
- With no terminal, the device flow is refused with exit 2 and a message saying
  it needs one to show a code for a browser (from
  crates/meow-cli/src/wire.rs:336, high).

## How I will know it was realised

`an_oauth_login_needs_a_terminal` in `crates/meow-cli/tests/auth.rs` passes:
`meow auth login copilot` with standard input closed takes the device path and
refuses (from crates/meow-cli/tests/auth.rs:299, high). No test checks that a
name other than `copilot` takes the key path in the same situation (reasoned
from crates/meow-cli/tests/auth.rs:299, low).

## What this does not settle

- Whether the comment at crates/meow-cli/src/wire.rs:93, which says the kind
  comes from the workspace when there is one, or the code, which reads only the
  name, is the intended behaviour (from crates/meow-cli/src/wire.rs:93, high).
- How a provider of kind `copilot` under another name gets its grant (reasoned
  from crates/meow-cli/src/wire.rs:1431, low).
