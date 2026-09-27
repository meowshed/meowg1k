---
id: ADR-0415
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2478, REQ-2479]
supersedes: []
---

# 0415. References resolve after every declaration file has been evaluated

## Decision

A reference by name resolves after every file has been evaluated, not as each
declaration is made (from https://github.com/meowshed/meowg1k/pull/121, high).
This covers names only: `meow.command` takes a value, so the tool or agent it
exposes must be bound before the line that exposes it, as any Starlark name
must (from https://github.com/meowshed/meowg1k/pull/123 and
crates/meow-star/src/declare.rs, high).

Once accepted, an agent can name a tool declared later in the same file or in
another file, and REQ-2479 holds as amended on 2026-09-27. What still doesn't
work: a dangling reference fails without a file, line or column
(crates/meow-star/src/error.rs, medium).

## Why

Declaration order inside and between files then stops mattering (from
https://github.com/meowshed/meowg1k/pull/121, high). Resolving as each
declaration is made would mean an agent couldn't name a model declared below
it, which is a rule nobody would guess from reading a file top to bottom (from
crates/meow-star/src/registry.rs:349-356, high). Markdown agents under
`.meow/agents/` are read after `meow.star` has been evaluated, so a reference
to one can only be checked once they are all in (from
crates/meow-star/src/loader.rs:410-417, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: resolve each reference at first use | Loading does no cross-checking, and a workspace with a broken agent still runs its other commands (reasoned from crates/meow-star/src/registry.rs:361, low) | REQ-2478 requires a missing provider, model or tool to fail at load time and not at first use (from docs/spec/starlark.md [R-STAR-032], high) |
| Resolve each reference as its declaration is made | The error arises inside the evaluation of the line that names the missing thing, so it can carry that line's location (reasoned from crates/meow-star/src/error.rs:57-66, low) | Declaration order inside and between files would matter (from https://github.com/meowshed/meowg1k/pull/121, high) |

## What it costs

The registry has to keep every reference as a name until the end, and
`Registry::resolve` walks all of them once after the last file (from
crates/meow-star/src/loader.rs:410-417, high). Because the check runs outside
any evaluation, its `StarError::Unknown` carries the kind, the name and the
closest match but no file or line, and the origin recorded for a referenced
agent is discarded (from crates/meow-star/src/error.rs:57-66 and
crates/meow-star/src/registry.rs:423-433, high). Only the first missing
reference is reported, because `resolve` returns on the first failure (from
crates/meow-star/src/registry.rs:361-440, high).

## What would reverse it

- A requirement makes declaration order significant, for example by requiring
  a name to be declared before the line that uses it, which would contradict
  REQ-2479 (reasoned from docs/spec/starlark.md [R-STAR-032], low).

## Consequences

- `meow.agent_named` records a reference and returns a value at once, and the
  name is checked only after every markdown agent has been read (from
  crates/meow-star/src/declare.rs:303-317, high).
- `meow.command` accepts a name as well as a value, and `resolve` checks that
  each command names a declared tool or agent (from
  crates/meow-star/src/value.rs:115-123 and
  crates/meow-star/src/registry.rs:436-450, high).

## How I will know it was realised

1. `a_reference_resolves_whatever_the_declaration_order` in
   crates/meow-star/tests/loading.rs passes: an agent naming a model declared
   below it loads, and a misspelt model fails at load time.
2. `naming_an_agent_that_does_not_exist_fails_at_load_time` in
   crates/meow-star/tests/agents.rs passes, suggesting the closest markdown
   agent's name.

## What this does not settle

- `meow.command` later took the tool or agent value rather than its name, so
  the thing it exposes has to exist before the line exposing it (from
  https://github.com/meowshed/meowg1k/pull/123, high). REQ-2479 now states both
  rules: a reference by name is resolved at the end, and a call taking a value
  needs that value bound first, as any Starlark name does (from
  docs/requirements/REQ-2479-references-resolved-after-all-declarations.md,
  high).
- Whether an unresolved reference must carry the file and line of the
  declaration that made it: REQ-2526 requires a location on every load-time
  error, and `StarError::Unknown` has none (from
  crates/meow-star/src/error.rs:57-66, high).
