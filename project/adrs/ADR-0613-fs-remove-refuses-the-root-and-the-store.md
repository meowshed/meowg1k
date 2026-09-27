---
id: ADR-0613
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2537, REQ-2538]
supersedes: []
---

# 0613. `fs.remove` refuses the workspace root and the store

## Decision

`fs.remove` refuses the workspace root and every path inside `.meow/.data/`,
saying the path isn't something it will delete (from
https://github.com/retran/meowg1k/pull/135 and
crates/meow-star/src/capability.rs:207-216, high).

Once this is accepted, both refusals work, and every other path inside the
workspace is removed, a directory with everything under it (from
crates/meow-star/src/capability.rs:218-223, high).

## Why

A handler that means to delete its own project can say so with a command,
where the intent is unmistakable (from
https://github.com/retran/meowg1k/pull/135, high). Confinement isn't policy,
because policy governs what a model decided and a handler is the workspace
author's own code; a path that escapes is a mistake whoever made it, and the
cheapest place to notice is before the call (from
https://github.com/retran/meowg1k/pull/135, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: `fs.remove` takes any path inside the workspace | One rule for every `fs` call, the workspace boundary (reasoned from crates/meow-star/src/capability.rs:44, low) | One call with `"."` or `".meow/.data"` deletes the project or its sessions and store (reasoned from crates/meow-star/src/capability.rs:218-223, low) |
| Put `fs` behind the policy layer and let a rule protect the store | The workspace chooses what is protected (reasoned from https://github.com/retran/meowg1k/pull/135, low) | It asks the author to approve their own script, a prompt nobody reads (from https://github.com/retran/meowg1k/pull/135, high) |

## What it costs

A handler that wants to clear the store or the project runs a shell command
(from https://github.com/retran/meowg1k/pull/135, high).

## What would reverse it

- A handler needs to delete part of `.meow/.data/`, such as a cache directory,
  and no `meow` command does it (reasoned from
  crates/meow-star/src/capability.rs:211, low).

## Consequences

- `fs.write`, `fs.append` and `fs.mkdir` still reach inside `.meow/.data/`;
  only removal is refused (from crates/meow-star/src/capability.rs:89-126 and
  :188-198, high).

## How I will know it was realised

1. `fs_remove_will_not_take_the_workspace_or_the_store` in
   crates/meow-star/tests/running.rs passes (from
   crates/meow-star/tests/running.rs:885, high). It removes `.meow/.data`
   only; a second case removing `"."` would cover REQ-2537.

## What this does not settle

- Whether writes into `.meow/.data/` should be refused as well (from
  crates/meow-star/src/capability.rs:89-126, high).
