---
id: REQ-2263
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2263

The binary MUST NOT read or migrate a v0.2.x session store, which v0.2.x kept at
`.meowg1k/.data/project.db`.

(from docs/design/0.3.0-sessions.md section 11, crates/meow-store/src/lib.rs:63
and v0.2.1:internal/adapters/sqlite/path/service.go:64, high)
