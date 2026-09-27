---
id: REQ-1827
artifact: requirement
topic: pkg
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1827

A fetch MUST fail when the unpacker declines to write an entry, and never pin a
package without it.

(from https://github.com/meowshed/meowg1k/pull/162 and
crates/meow-cli/src/fetch.rs:238, high)
