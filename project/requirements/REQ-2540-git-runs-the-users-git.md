---
id: REQ-2540
artifact: requirement
topic: star
class: functional
status: approved
revised: 2026-09-27
elaborates:
verification: behavioural
---

# REQ-2540

`@std//git` MUST run the user's own `git` program as a subprocess, so it reads
the repository with the configuration, hooks and worktrees that program sees.

(from https://github.com/retran/meowg1k/pull/136 and
crates/meow-star/src/capability_git.rs:25, high)
