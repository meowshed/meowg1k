---
id: BUG-0117
artifact: bug
status: approved
severity: minor
found: 2026-09-27
revised: 2026-09-27
issue: 240
---

# The session identifier repeats 16 bits it counts as entropy

The identifier's comment says it holds 80 bits of entropy from the clock and
the process id. The binary fills the last two of those ten bytes with the low
16 bits of the nanoseconds, which bytes 2 and 3 already hold, so the
identifier carries at most 64 distinct bits, and the short form's 40 bits are
24 bits of process id and those 16 repeated bits.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments.

1. Read `entropy` at `crates/meow-cli/src/wire.rs:928-938`.
2. Compare bytes 2 and 3 of the result, which are the low half of
   `nanos.to_be_bytes()`, with bytes 8 and 9, which are
   `(nanos as u16).to_be_bytes()`. They're equal for every call.

## What the system does

`out[..4]` is the 32-bit sub-second nanosecond count, `out[4..8]` the process
id, and `out[8..]` the low 16 bits of the same nanosecond count. The comment at
`crates/meow-core/src/id.rs:18-19` says "80 bits of entropy from the clock and
the process id", and `SessionId::new`'s comment says "80 bits of entropy".

## What it should do, and why

No requirement sets a number of bits. REQ-2220 says the short identifier "comes
from the identifier's entropy part, the clock's sub-millisecond digits and the
process id, so it distinguishes sessions the timestamp cannot". The comment
should state what the bytes hold, and the last two bytes should carry
information the others don't, such as a per-process counter or random bytes.

## Triage

The defect enters in `meow-cli`'s `entropy` and in the `meow-core` comment that
describes it. ADR-0438 chose the clock and the process id. It's minor: the
binary opens one session per process, so two sessions from one process never
compete for a short identifier today, and an ambiguous short identifier is
refused with the candidates listed (REQ-2221).

## Closed by

A unit test beside `entropy`, named for example
`entropy_bytes_do_not_repeat_the_clock`, that expects bytes 8 and 9 to differ
from bytes 2 and 3 for two calls in one process, and the comment in `id.rs`
corrected in the same change.
