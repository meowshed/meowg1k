---
id: BUG-0037
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue: 210
---

# Two decisions name a `meow session resume` subcommand the binary doesn't have

ADR-0109 and ADR-0111 give `meow session resume <id>` as a way to continue a
session. The `session` group has `list`, `show`, `fork`, `export` and `gc`, and
`fork` has no `--model` flag, both of which the design the decisions quote
promised.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Run `meow session resume x`. It prints `error: unrecognized subcommand
   'resume'`. This was run against `target/debug/meow`.
2. Read `crates/meow-cli/src/surface.rs:167-257`, which declares the `session`
   subcommands and their arguments.
3. Read `project/adrs/ADR-0109-no-chat-interface.md:17` and
   `project/adrs/ADR-0111-fresh-session-by-default.md:37`, which name
   `meow session resume`.

ADR-0436 (line 23) and ADR-2204 (line 22) already record that `resume` and
`fork --model` don't exist.

## What the system does

The binary resumes a session only through `--continue` on the command that
wrote it, and forks without changing the model.

## What it should do, and why

No requirement asks for `meow session resume` or for `fork --model`; searching
`project/requirements/` for either finds nothing. The defect is that two
decisions describe a command that doesn't exist, which `CLAUDE.md`'s
`docs_must_match_code` principle forbids.

## Triage

The defect enters in the decision records, carried over from the deleted
`docs/design/`. It is minor. Both decisions are still drafts, so the fix is to
edit them to name `--continue` only, or to add a requirement and code for
`resume` if the project wants it.

## Closed by

`meow-method check` passing with no decision naming a subcommand that
`crates/meow-cli/src/surface.rs` doesn't declare; no code test applies.
