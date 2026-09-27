---
id: ADR-0465
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1222, REQ-1229]
supersedes: []
---

# 0465. The trust question lists the packages a workspace declares

## Decision

The trust question shows declared packages, and adding a package withdraws an
agreement as adding a tool does (from
https://github.com/retran/meowg1k/pull/163, high).

Once this is accepted, `Declared::of` adds one line per declared package,
`package <name> <version> from <source>`, to the lines the trust question shows
and the fingerprint covers (from crates/meow-cli/src/trust.rs:43, high). A
changed name, version or source therefore changes the fingerprint and asks
again (reasoned from crates/meow-cli/src/trust.rs:83, medium). A change to a
package's contents that keeps its version and source doesn't, because the
fingerprint doesn't cover the lockfile's hash (from
crates/meow-cli/src/trust.rs:44, high).

## Why

A workspace that declares a package will run code somebody else wrote, the most
important line on that list, and leaving it off made the prompt ask less than it
appeared to (from https://github.com/retran/meowg1k/pull/163, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: leave packages off the trust question | A shorter question, and adding a package doesn't withdraw an agreement (reasoned from crates/meow-cli/src/trust.rs:33, low) | The prompt asks less than it appears to (from https://github.com/retran/meowg1k/pull/163, high) |

The code and the history name no other option: the question is whether a
package is on the list, and the only other answer is the one that lost.

## What it costs

Adding a package, or changing its version or source, withdraws the agreement,
so every machine that trusted the workspace is asked again (from
crates/meow-cli/tests/trust.rs:290, high). The question grows by one line per
package (from crates/meow-cli/src/trust.rs:43, high).

## What would reverse it

A package that can't reach anything the workspace's policy doesn't already
allow would make it no wider an authority than a local file, and so not worth a
line; today a package runs under the same rules as the workspace's own code
(reasoned from
docs/requirements/REQ-1824-package-code-follows-workspace-rules.md, low).

## Consequences

- What is shown is what is agreed to, and a package was missing from what was
  shown (from https://github.com/retran/meowg1k/pull/163, high).
- The lines are sorted, so reordering declarations doesn't ask again (from
  crates/meow-cli/src/trust.rs:66, high).
- `meow pkg` is exempt from the trust gate, so a person can see what a
  workspace would pull in before deciding to trust it (from
  https://github.com/retran/meowg1k/pull/163, high).

## How I will know it was realised

`a_declared_package_is_shown_in_the_question` and
`adding_a_package_asks_again` in `crates/meow-cli/tests/trust.rs` pass (from
crates/meow-cli/tests/trust.rs:268, high).

## What this does not settle

- REQ-1222 lists agents, tools and policy as what the question shows, and
  doesn't name packages, so the requirement is narrower than the code (from
  docs/requirements/REQ-1222-untrusted-workspace-shows-declarations.md, high).
- Whether a new lockfile hash under an unchanged version should ask again (from
  crates/meow-cli/src/trust.rs:44, high).
