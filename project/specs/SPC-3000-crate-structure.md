---
id: SPC-3000
artifact: spec
status: live
revised: 2026-09-27
checked-at:
states: [REQ-3000, REQ-3001, REQ-3002, REQ-3003, REQ-3004, REQ-3005, REQ-3006, REQ-3007, REQ-3008, REQ-3009, REQ-3010, REQ-3011]
---

# The workspace's crate structure

## Scope

This covers which of the ten workspace crates may depend on which, and what
`meow-core` may do (from CLAUDE.md crate_boundaries and
docs/design/0.3.0-architecture.md section 4, high). It doesn't cover what each
crate does for a user, which the other specifications state, or external
dependencies, which `deny.toml` governs.

## Boundary

| Surface | What it is |
| --- | --- |
| `crates/*/Cargo.toml` | Each crate's `[dependencies]` and `[dev-dependencies]` on other workspace crates |
| `cargo metadata` | The dependency graph the manifests produce |

(from Cargo.toml and crates/*/Cargo.toml, high)

## Behaviour

### The base

- `meow-core` depends on no other workspace crate [REQ-3000] and performs no
  input or output [REQ-3001].

### Storage and sessions

- `meow-store` depends on no workspace crate but `meow-core` [REQ-3002].
- `meow-session` depends on no workspace crate but `meow-store` and
  `meow-core` [REQ-3003].
- `meow-index` depends on no workspace crate but `meow-store` and `meow-core`
  [REQ-3008].

### Models, policy and the engine

- `meow-llm` depends on no workspace crate but `meow-core` [REQ-3004].
- `meow-policy` depends on no workspace crate but `meow-core` [REQ-3005].
- `meow-agent` depends on no workspace crate but `meow-llm`, `meow-policy` and
  `meow-core` [REQ-3006], and never on `meow-star`, directly or through
  another crate [REQ-3007].

### The surface and the binary

- `meow-star` depends on no workspace crate but `meow-agent` and the crates
  `meow-agent` may depend on, and reaches the terminal, the session log and the
  index through traits [REQ-3009].
- `meow-ui` depends on no workspace crate but `meow-core` [REQ-3010].
- `meow-cli` may depend on every other workspace crate [REQ-3011].

## Failure paths

| Condition | What happens |
| --- | --- |
| A manifest names a workspace crate its row doesn't permit | The build succeeds, because no check reads the graph; the crossing breaks REQ-3000 to REQ-3010 and is found in review (from deny.toml and Cargo.toml, high) |
| `meow-core` gains a dependency that performs input or output | The build succeeds, and the crate breaks REQ-3001 (from crates/meow-core/Cargo.toml, high) |
