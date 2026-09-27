---
id: ADR-0458
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1229]
supersedes: []
---

# 0458. The trust fingerprint covers declarations, not handler bodies

## Decision

The fingerprint is SHA-256 over the sorted agent, tool, command and policy lines
the person read, so widening a policy or adding a tool asks again and editing a
handler's body doesn't (from https://github.com/retran/meowg1k/pull/154, high).

Once this is accepted, `Declared::of` records one line per agent, package, tool,
command and policy rule, sorted, and the fingerprint is SHA-256 over those
lines; a workspace with no policy records "policy none, so every tool call is
asked" (from crates/meow-cli/src/trust.rs:33 and
crates/meow-cli/src/trust.rs:85, high). Packages were added to the lines after
this choice, so the lines come in five kinds, not four (from
crates/meow-cli/src/trust.rs:43 and
https://github.com/retran/meowg1k/pull/163, high). A comment, a handler's body,
a model's name and an agent's system prompt change nothing the fingerprint
covers (from crates/meow-cli/src/trust.rs:80, high).

## Why

Re-asking on every edit makes the prompt a formality people click through (from
https://github.com/retran/meowg1k/pull/154, high). Trust is about authority, and
a handler's body can't widen authority without widening one of the four recorded
things (from https://github.com/retran/meowg1k/pull/154, high).

The second reason doesn't hold against the code. A handler calls `@std//shell`,
`@std//fs` and `@std//http` directly, and ADR-2002 keeps those calls outside the
policy layer, so an edited handler body can run a new program or reach a new
host without changing any recorded line (from
crates/meow-star/src/capability.rs:229,
crates/meow-star/src/capability_http.rs:19 and
docs/adrs/ADR-2002-policy-judges-model-tool-calls.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Ask again on every edit to `.meow/` | Catches every change, including a handler body that calls `shell.run` or `http.post` for the first time (reasoned from crates/meow-star/src/capability.rs:229, low) | The prompt becomes a formality people click through (from https://github.com/retran/meowg1k/pull/154, high) |
| Do nothing: trust a path once and keep it whatever changes | One question per workspace, ever (reasoned from docs/spec/auth.md Decisions, low) | REQ-1229 forbids a workspace whose declarations changed keeping its trust silently (from docs/spec/auth.md [R-AUTH-034], high) |
| Key the fingerprint by content, not path | Moving a workspace would keep its trust (from https://github.com/retran/meowg1k/pull/154, high) | A repository trusted anywhere would be trusted everywhere (from https://github.com/retran/meowg1k/pull/154, high) |

## What it costs

An edit to a handler's body runs without a new question, including one that adds
a shell command or an HTTP request the person never saw (from
crates/meow-cli/tests/trust.rs:186 and crates/meow-star/src/capability.rs:229,
high). The question shows names, not what a tool or a handler does, so the
person agrees to a list of names (from crates/meow-cli/src/trust.rs:33, high).

## What would reverse it

A handler body is observed to do something the person didn't agree to without
changing a recorded line, for example a new `shell.run` call, and that is judged
a trust failure; recording which `@std//` modules the workspace loads would then
be the smallest widening of the fingerprint (reasoned from
crates/meow-cli/src/trust.rs:33, low).

## Consequences

- Reordering a declaration isn't a change, because the lines are sorted (from
  crates/meow-cli/src/trust.rs:67, high).
- Widening a rule from `ask` to `allow`, adding a tool or adding a package asks
  again (from crates/meow-cli/tests/trust.rs:130 and
  crates/meow-cli/tests/trust.rs:290, high).
- The lines are the same text the person is shown, so what is recorded is what
  was shown (from crates/meow-cli/src/trust.rs:23, high).

## How I will know it was realised

Four tests in `crates/meow-cli/tests/trust.rs` pass:
`widening_the_policy_withdraws_the_agreement`,
`a_new_tool_withdraws_the_agreement`, `adding_a_package_asks_again` and
`editing_a_handler_does_not_ask_again` (from crates/meow-cli/tests/trust.rs:130,
high).

## What this does not settle

- ADR-1202 records that a change to the declarations withdraws trust. This
  decision adds only what counts as a change.
- Whether the `@std//` modules a workspace loads belong in the fingerprint,
  since a handler reaches them without policy (from
  crates/meow-star/src/capability.rs:9, high).
