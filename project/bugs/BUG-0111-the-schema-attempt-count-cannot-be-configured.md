---
id: BUG-0111
artifact: bug
status: approved
severity: minor
violates: REQ-1644
found: 2026-09-27
revised: 2026-09-27
issue: 234
---

# The schema retry count is a constant, not a configured value

A response that fails schema validation is retried up to `SCHEMA_ATTEMPTS`,
which is the constant 3, and nothing in a declaration, a request or an agent's
front matter changes it.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Search the tree: `grep -rn SCHEMA_ATTEMPTS crates`. It's defined at
   `crates/meow-llm/src/schema.rs:9` and read by the providers, for example
   `crates/meow-llm/src/anthropic.rs:18`.
2. Look for a keyword on `meow.agent`, `meow.model` or the agent front matter
   that sets it. There is none.

## What the system does

`pub const SCHEMA_ATTEMPTS: u32 = 3;` at `crates/meow-llm/src/schema.rs:9`.
Every provider retries against that value.

## What it should do, and why

REQ-1644: a response that fails validation "MUST be retried with the
validation error included in the follow-up request, up to a configured attempt
count." A person should be able to set the count, or the requirement should say
the count is fixed.

## Triage

The defect enters in `meow-llm`. It's minor: the retry happens and the count is
reasonable, and only the configuring is missing. The requirement may be what's
wrong here: if nobody needs to change the count, REQ-1644 should say "up to
three attempts" and this record closes by amending it.

## Closed by

Either a test in `crates/meow-llm/tests/`, named for example
`the_schema_attempt_count_comes_from_the_request`, that sets two attempts and
counts two requests, or an amendment to REQ-1644 naming the fixed count with
`SCHEMA_ATTEMPTS` cited.
