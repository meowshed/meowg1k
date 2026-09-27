---
id: REQ-3009
artifact: requirement
topic: architecture
class: non-functional
status: approved
revised: 2026-09-27
elaborates:
verification: static
---

# REQ-3009

`meow-star` MUST NOT depend on a workspace crate other than `meow-agent` and the
crates `meow-agent` may depend on. What it needs from the terminal, the session
log and the index arrives as a trait, so a test drives a handler with no
terminal and no account.

(from CLAUDE.md crate_boundaries, docs/design/0.3.0-architecture.md section 4
and crates/meow-star/Cargo.toml, high)
