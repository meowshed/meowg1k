---
id: BUG-0135
artifact: bug
status: approved
severity: minor
violates: [REQ-3000, REQ-3002, REQ-3003, REQ-3004, REQ-3005, REQ-3006, REQ-3007, REQ-3008, REQ-3009, REQ-3010]
found: 2026-09-27
revised: 2026-09-27
issue: 258
---

# Nothing enforces the crate dependency graph

The requirements fix which workspace crate may depend on which, and no test,
lint or `cargo deny` rule checks it, so an edge such as BUG-0134 lands and
`mise run all` stays green.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Read `deny.toml`'s `[bans]` section, lines 62-72. It sets
   `multiple-versions`, `wildcards` and `allow-wildcard-paths`, and bans no
   crate.
2. Search for a graph check: `grep -rln "cargo_metadata\|cargo metadata"
   crates` finds nothing.
3. Read `[tasks.all]` in `mise.toml`, line 98. It runs `fmt-check`, `check`,
   `test`, `doc`, `deny`, `unused-deps` and `lint-md`, and none of them reads
   the graph, so the edge BUG-0134 describes passes the gate.

## What the system does

The graph is kept by review alone. Every requirement REQ-3000 to REQ-3011 has
`verification: static`, and no static check runs.

## What it should do, and why

REQ-3000 to REQ-3010 each forbid edges, for example REQ-3003: "`meow-session`
MUST NOT depend on a workspace crate other than `meow-store` and
`meow-core`." A static check should fail the gate when a crate gains a
forbidden edge.

## Triage

The defect enters in the gate, `mise run all`, which has no step for it. It's
minor because the graph holds except for BUG-0134; the missing guard is why
that edge went unnoticed. A test reading `cargo metadata` and comparing each
crate's workspace dependencies with an allow list, or `[bans]` entries with
`wrappers`, would fix it.

## Closed by

A test, named for example `the_crate_graph_matches_the_requirements`, run by
`mise run test`, that fails on the `meow-session` to `meow-policy` edge.
