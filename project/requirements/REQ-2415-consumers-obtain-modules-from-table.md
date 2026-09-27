---
id: REQ-2415
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: judgement
verifier: person
---

# REQ-2415

Every consumer MUST obtain a module from the one table runtime modules are
registered in. No code path may construct a context or module set of its own.
(from docs/spec/starlark.md [R-STAR-010], high)

The judge is the repository owner, who approves every pull request under
`CLAUDE.md`'s `reviewable_history`, and who checks that a change adding a
runtime module registers it in `crates/meow-star/src/modules.rs` and that a
handler context is built only in `crates/meow-star/src/context.rs`. (from
`CLAUDE.md` `one_context_builder`, high)
