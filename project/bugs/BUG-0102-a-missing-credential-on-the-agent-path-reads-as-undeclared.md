---
id: BUG-0102
artifact: bug
status: approved
severity: major
violates: [REQ-1201, REQ-2852]
found: 2026-09-27
revised: 2026-09-27
issue:
---

# A missing credential on the agent path reads as an undeclared provider and exits 9

When a handler runs an agent whose provider has no credential, the error says
the provider "is not declared" and the process exits 9. It should name the
three places a credential is looked for and exit 6.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments; `cargo build -p meow-cli`, `HOME` set to
an empty scratch directory, `ANTHROPIC_API_KEY` unset.

1. Create `.meow/meow.star`:

   ```python
   meow.provider(name = "anthropic", kind = "anthropic")
   meow.model(name = "smart", provider = "anthropic", id = "claude-sonnet-4-5", context = 200000, max_output = 1000)
   helper = meow.agent_named("helper")
   def _go(ctx):
       r = helper.run("hi")
       return "true"
   meow.command(meow.tool(name = "go", about = "go", run = _go))
   ```

   and `.meow/agents/helper.md` with `model: smart` in its front matter.
2. Run `meow trust`, then `meow go`.

## What the system does

It prints ``error: provider `anthropic` is not declared`` with a traceback at
`meow.star:5` and exits 9. The binary builds engines only for providers that
have a credential, and `Runtime::engine_for` reports a missing engine as
`StarError::Unknown { kind: "provider" }`,
`crates/meow-star/src/run.rs:263-269`. The message REQ-1201 asks for exists as
`no_credential` at `crates/meow-cli/src/wire.rs:1456` and isn't used on this
path.

## What it should do, and why

REQ-1201: a provider with no credential "MUST fail naming all three places,
with the environment variable spelled out". REQ-2852 maps a provider or
credential failure to exit code 6. The run should print the `no_credential`
text and exit 6.

## Triage

The defect enters where `meow-star` looks up the engine for a model and can't
tell a missing credential from a missing declaration. ADR-0429 records that a
provider without a credential is skipped at startup. It's major because both
requirements' main behaviour fails on the path every agent-running command
takes, and the message sends the reader to fix a declaration that is correct.

## Closed by

A test in `crates/meow-cli/tests/`, named for example
`an_agent_whose_provider_has_no_credential_exits_6`, that runs the workspace
above with no key and expects exit 6 and stderr naming `api_key`,
`meow auth login anthropic` and `$ANTHROPIC_API_KEY`.
