---
id: REQ-1826
artifact: requirement
topic: pkg
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-1826

A fetch MUST refuse an archive holding an entry whose path contains `..` or
starts at a root.

(from https://github.com/meowshed/meowg1k/pull/162 and
crates/meow-cli/src/fetch.rs:216, high)
