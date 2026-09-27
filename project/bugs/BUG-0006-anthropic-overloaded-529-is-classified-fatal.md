---
id: BUG-0006
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue:
---

# Anthropic's 529 "overloaded" is classified `Fatal`

`LlmError::class` lists 408, 429, 500, 502, 503 and 504 as `Transient` and
everything else as `Fatal`. Anthropic answers 529 when its API is overloaded,
which is a transient condition, so once retry is wired (BUG-0005) a 529 will
still fail the run on the first attempt.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Build `LlmError::Http { status: 529, quota_exhausted: false, .. }`.
2. Call `.class()` on it. It returns `Class::Fatal`.

## What the system does

`crates/meow-llm/src/error.rs:133-136` matches `408 | 429 | 500 | 502 | 503 |
504` to `Transient` and sends every other status to `Fatal`. The Anthropic
provider builds its error from the status as it arrives
(`crates/meow-llm/src/anthropic.rs:161-180`), so a 529 reaches `class`
unchanged.

## What it should do, and why

No requirement covers 529. REQ-1625 lists the transient codes and REQ-1626 the
fatal ones, and 529 is in neither list, so the code's choice of `Fatal` is a
default nobody made. This is a gap in the requirements and routes to the
requirements step: REQ-1625 should either name 529 or let a provider mark its
own vendor-specific transient codes.

## Triage

The defect enters in `LlmError::class`. It is minor, because no call is retried
today (BUG-0005), so a 529 fails the run whichever class it has. It becomes a
visible failure as soon as retry is wired.

## Closed by

A test in `crates/meow-llm/tests/spec.rs`, named for example
`anthropic_overloaded_is_transient`, that expects `Class::Transient` for an
Anthropic 529.
