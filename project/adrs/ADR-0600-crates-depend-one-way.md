---
id: ADR-0600
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-3000, REQ-3001, REQ-3002, REQ-3003, REQ-3004, REQ-3005, REQ-3006, REQ-3007, REQ-3008, REQ-3009, REQ-3010, REQ-3011]
supersedes: []
---

# 0600. The workspace's crates depend one way, as CLAUDE.md's table sets out

## Decision

Each crate depends only on the crates its row in CLAUDE.md's `crate_boundaries`
table names, on what those crates may depend on, and on `meow-core`; `meow-core`
depends on no workspace crate and performs no input or output (from CLAUDE.md
crate_boundaries and docs/design/0.3.0-architecture.md section 4, high). I read
a row as "these crates and everything below them" because every crate that uses
the domain types names `meow-core` directly while the table leaves it implied,
and reading the rows literally would make `meow-star`, `meow-agent` and
`meow-session` violate them for that alone (reasoned from crates/*/Cargo.toml,
low).

Once this is accepted, nine of the ten manifests keep to it (from
crates/*/Cargo.toml, high). `meow-session` doesn't: it depends on `meow-policy`
to use the `REDACTED` placeholder in export, which REQ-3003 forbids (from
crates/meow-session/Cargo.toml and crates/meow-session/src/export.rs:230, high).
Nothing checks the graph mechanically: section 4 says `cargo-deny` and a
workspace lint enforce it, and `deny.toml` names no workspace crate (from
docs/design/0.3.0-architecture.md section 4 and deny.toml, high).

## Why

v0.2.x broke the equivalent boundary when `internal/core/starlark` imported
`internal/adapters/gateway` for an embeddings factory, and section 4 says that
must not be reproduced (from docs/design/0.3.0-architecture.md section 4 and
CLAUDE.md crate_boundaries, high). Keeping `meow-agent` free of Starlark lets
the engine be tested without a script and would let a second frontend drive it
(from docs/design/0.3.0-architecture.md section 4, high). Keeping `meow-ui` to
the event types lets a renderer run from a recorded log (from CLAUDE.md
crate_boundaries, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: let any crate import any other, as v0.2.x's packages did | No trait or port is needed to reach a crate above, so a feature lands in one place (reasoned from docs/design/0.3.0-architecture.md section 4, low) | v0.2.x's Starlark package imported the model gateway, and the engine could then not be tested without a script (from docs/design/0.3.0-architecture.md section 4, high) |
| A finer split: a `meow-tools` crate for the built-in tools and a `meow-provider-*` crate per vendor | A vendor or a tool compiles and is reviewed alone (reasoned from docs/design/0.3.0-architecture.md section 4, low) | Dropped between the design and CLAUDE.md with no reason written down; the tools became `@std//` modules in the one table in `meow-star`, and CLAUDE.md requires that one table (reasoned from CLAUDE.md one_context_builder and crates/meow-star/src/modules.rs, low) |

## What it costs

Whatever `meow-star` needs from a crate above it, such as the terminal, the
session log or the index, has to arrive as a trait that `meow-cli` implements,
so each such need adds a port (from CLAUDE.md crate_boundaries and
crates/meow-star/src/port.rs, high). A constant shared by two crates on
different branches has to move down into `meow-core`, which is the fix
`meow-session`'s use of `meow_policy::REDACTED` now needs (reasoned from
crates/meow-session/src/export.rs:230, low).

## What would reverse it

- A second binary or frontend needs a crate arrangement the table forbids, and
  the table is rewritten for it (reasoned from docs/design/0.3.0-architecture.md
  section 4, low).

## Consequences

- REQ-3000 to REQ-3011 state each row, and SPC-3000 collects them (from this
  decision, high).
- `meow-session`'s dependency on `meow-policy` is a defect against REQ-3003;
  moving `REDACTED` into `meow-core`, or giving it to the exporter as the
  redaction it already takes from its caller, removes it (reasoned from
  crates/meow-session/src/export.rs:230 and ADR-0435, low).
- Section 4's claim that `cargo-deny` and a lint enforce the layering is false
  today, so a crossing is caught only in review (from deny.toml, high).

## How I will know it was realised

1. `cargo metadata --format-version 1 --no-deps` lists, for each crate, only
   the workspace dependencies its REQ permits, `meow-session` included (from
   crates/*/Cargo.toml, high).
2. `grep -rn 'std::fs\|std::net\|std::io' crates/meow-core/src` finds nothing
   (from crates/meow-core/src, high).

## What this does not settle

- Which tool enforces the graph: a `cargo-deny` ban, a test over `cargo
  metadata` or review (from deny.toml, high).
- Whether development dependencies count: `meow-index` names `meow-core` only
  as a development dependency, which the REQs permit either way (from
  crates/meow-index/Cargo.toml, high).
