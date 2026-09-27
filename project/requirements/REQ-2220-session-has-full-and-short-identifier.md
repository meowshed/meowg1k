---
id: REQ-2220
artifact: requirement
topic: session
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2220

Every session MUST have a full identifier that sorts by creation time, and a
short identifier that is its last eight characters. The tail comes from the
identifier's entropy part, the clock's sub-millisecond digits and the process id,
so it distinguishes sessions the timestamp cannot (from
crates/meow-cli/src/wire.rs, high). The length is fixed rather
than "shortest unique" so that an identifier written into a commit message keeps
resolving as sessions accumulate.

(from docs/spec/session.md [R-SESSION-034], high)
