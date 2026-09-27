---
id: ADR-0435
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2260]
supersedes: []
---

# 0435. Export takes its redaction from the caller

## Decision

`Redaction` is supplied by the caller, not derived by `meow-session`, and the
binary is meant to pass `Policy::sensitive_for` (from
https://github.com/meowshed/meowg1k/pull/128, high).

Once this is accepted, `Sessions::export_json` and `Sessions::export_markdown`
replace the value of every argument the caller names, in both formats (from
crates/meow-session/src/export.rs:19, high). The binary doesn't pass the
policy's list yet: `meow session export` builds a `Redaction` with an empty
`arguments` list, so nothing is redacted through the command line (from
crates/meow-cli/src/wire.rs:713, high).

## Why

Which arguments are sensitive is a property of the policy and the tool set, and
`meow-session` knows neither (from https://github.com/meowshed/meowg1k/pull/128,
high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Derive the redaction inside `meow-session` | A caller couldn't forget to pass the list, and every export would be redacted without the binary wiring anything (reasoned from crates/meow-cli/src/wire.rs:713, low) | `meow-session` knows neither the policy nor the tool set (from https://github.com/meowshed/meowg1k/pull/128, high) |
| Redact the arguments before they reach the log, so export needs no list | The log would never hold a marked value, which REQ-2043 asks for anyway (reasoned from docs/requirements/REQ-2043-sensitive-value-never-logged-clear.md, low) | The engine records the arguments it was given and `meow-star` writes them unchanged, so export is where the redaction happens today (from crates/meow-session/src/export.rs:196 and crates/meow-star/src/run.rs:710, medium) |
| Do nothing: export the log as recorded | No policy has to be loaded to read a session, which is the state `meow session export` is in now (from https://github.com/meowshed/meowg1k/pull/129, high) | REQ-2260 requires every value the policy marked sensitive to be redacted in both formats (from docs/spec/session.md [R-SESSION-092], high) |

## What it costs

Every caller of export has to know the policy and pass its list, and a caller
that passes an empty list gets an unredacted export with no error (reasoned from
crates/meow-session/src/export.rs:219, low). The binary pays that cost now:
`meow session export` doesn't load a policy, so the requirement is met in
`meow-session` and not through the command line (from
https://github.com/meowshed/meowg1k/pull/129, high).

## What would reverse it

If the sensitive arguments were recorded in the log itself, for example as a
field on `EventKind::ToolCall`, `meow-session` could derive the redaction
without the policy, and the caller's list would be redundant (reasoned from
crates/meow-session/src/export.rs:196, low).

## Consequences

- `meow-session` stays independent of the policy's rules: it takes a list of
  argument names and replaces their values with `meow_policy::REDACTED` (from
  crates/meow-session/src/export.rs:219, high).
- The same `Redaction` carries whether reasoning is included, so one value
  describes everything an export leaves out (from
  crates/meow-session/src/export.rs:25, high).
- `meow session show` passes `Redaction::default()`, so it redacts nothing
  either (from crates/meow-cli/src/wire.rs:667, high).

## How I will know it was realised

`a_sensitive_value_is_redacted_in_both_formats` in
`crates/meow-session/tests/fork.rs` passes: an export given
`arguments: ["token"]` contains neither format's secret and keeps the rest (from
crates/meow-session/tests/fork.rs:340, high). The decision is realised end to
end when `meow session export` passes `Policy::sensitive_for` and a test drives
it through the binary; no such test exists yet (from
crates/meow-cli/src/wire.rs:713, high).

## What this does not settle

- `meow session export` passes an empty redaction, because the binary doesn't
  load a policy for a session command yet (from
  https://github.com/meowshed/meowg1k/pull/129, high).
- Whether the log should hold a marked value at all: the engine hands the tool
  arguments to `meow-star` unredacted and `meow-star` records them, which
  REQ-2043 forbids (from crates/meow-agent/src/engine.rs:287 and
  crates/meow-star/src/run.rs:710, medium).
