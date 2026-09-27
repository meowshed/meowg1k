---
id: REQ-3007
artifact: requirement
topic: architecture
class: non-functional
status: approved
revised: 2026-09-27
elaborates:
verification: static
---

# REQ-3007

`meow-agent` MUST NOT depend on `meow-star`, directly or through another crate,
so the engine is tested without a script.

(from CLAUDE.md crate_boundaries, docs/design/0.3.0-architecture.md section 4
and crates/meow-agent/Cargo.toml, high)
