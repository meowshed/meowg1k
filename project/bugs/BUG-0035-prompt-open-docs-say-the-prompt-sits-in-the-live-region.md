---
id: BUG-0035
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue:
---

# `Renderer::prompt_open`'s documentation says the prompt occupies the live region

The trait's doc comment and the doc comment of the test that checks it both say
an approval prompt lives in the live region. The terminal renderer commits the
prompt to the transcript, as REQ-2819 asks, and the test asserts that.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Read `crates/meow-ui/src/lib.rs:40-45`: "it occupies the live region and
   never overwrites the transcript above it."
2. Read the doc comment of `a_prompt_does_not_overwrite_the_transcript` at
   `crates/meow-ui/tests/renderers.rs:520-521`: "a prompt lives in the live
   region".
3. Read the same test's body at `crates/meow-ui/tests/renderers.rs:538-543` and
   `Tty::prompt_open` at `crates/meow-ui/src/tty.rs:401-414`: the prompt is
   inserted into the transcript, and the live region shows only "waiting for
   your answer".

## What the system does

The code commits the prompt to the transcript. The two doc comments describe
the design REQ-2819 replaced.

## What it should do, and why

No requirement governs a doc comment. `CLAUDE.md` asks that a document
describing something other than the code be fixed in the change that finds it
(`docs_must_match_code`). REQ-2819, "An approval prompt MUST be committed to the
transcript rather than drawn in the live region", is what both comments should
say.

## Triage

The defect enters in `meow-ui`'s documentation. It is minor: the behaviour is
right, and a reader of the trait is told the opposite. The fix is to rewrite
both comments to cite REQ-2819.

## Closed by

A review of the two comments; no test can check a doc comment. The existing test
`a_prompt_does_not_overwrite_the_transcript` already guards the behaviour.
