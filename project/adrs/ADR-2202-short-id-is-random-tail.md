---
id: ADR-2202
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2220]
supersedes: []
---

# 2202. A session's short identifier is the last eight characters of a time-sortable identifier

## Decision

Every session has a full identifier that sorts by creation time, and a short
identifier that is its last eight characters (from docs/spec/session.md
[R-SESSION-034], high).

Once this holds, the full identifier is 26 Crockford base32 characters, a
48-bit millisecond timestamp followed by 80 bits of entropy, and a person types
the eight-character tail (from crates/meow-core/src/id.rs:8-25, high). A
literal a person types resolves by matching the end of an identifier, so any
tail length works (from crates/meow-store/src/rows.rs:308-314, high).

## Why

Nobody types a UUID such as `meow show-session 9f8e7d6c-5b4a-...` (from
docs/spec/session.md, high). The tail of a time-sortable identifier is its
random part, so it distinguishes sessions the timestamp can't, and a fixed
length keeps an identifier written into a commit message resolving as sessions
accumulate (from docs/spec/session.md [R-SESSION-034], high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| UUIDs, as in v0.2.x (the do-nothing option) | Unique with no coordination and no clock, and already implemented (reasoned from docs/design/0.3.0-sessions.md section 2, low) | Nobody types `meow show-session 9f8e7d6c-5b4a-...` (from docs/spec/session.md, high) |
| An eight-character prefix of the time-sortable identifier, as the withdrawn [R-SESSION-030] required | Short identifiers that sort by creation time like the full ones (reasoned from docs/spec/session.md [R-SESSION-030], low) | The prefix is almost entirely timestamp, so two sessions created in the same period share it (from docs/spec/session.md, high) |
| The shortest unique prefix, extended on collision, as the design first sketched | Shorter identifiers while a workspace holds few sessions (from docs/design/0.3.0-sessions.md section 4, high) | An identifier written into a commit message stops resolving as sessions accumulate (from docs/spec/session.md, high) |

(from docs/spec/session.md, high)

## What it costs

The tail isn't guaranteed unique. The binary fills the entropy from the clock's
sub-second nanoseconds and the process identifier, not a random number
generator (from crates/meow-cli/src/wire.rs:928-938 and
https://github.com/retran/meowg1k/pull/129, high). The last eight characters
are the low 24 bits of the process identifier and the low 16 bits of the
nanoseconds, so two sessions minted in one process share a tail about once in
65,536 pairs, and a person who meets a shared tail types a longer suffix
(reasoned from crates/meow-cli/src/wire.rs:928-938 and
crates/meow-core/src/id.rs:36-53, low).

## What would reverse it

- Ambiguous short identifiers, the `SessionError::Ambiguous` error, showing up
  in ordinary use in one workspace (reasoned from
  crates/meow-session/src/resolve.rs:69-106, low).
- A need for identifiers that are unguessable, which the pull request that
  introduced the entropy names as the condition for changing it (from
  https://github.com/retran/meowg1k/pull/129, high).

## Consequences

- A short identifier that matches more than one session fails with the
  candidates listed and never picks one (from
  crates/meow-session/src/resolve.rs:69-106, high).
- `meow session list` prints the short form (from
  crates/meow-cli/src/session.rs:173-186, high).
- The identifier is minted by the caller from a clock and entropy passed in, so
  `meow-core` performs no input or output and a test can build a known
  identifier (from crates/meow-core/src/id.rs:32-35, high).

## How I will know it was realised

1. `the_short_form_is_the_random_tail` in crates/meow-core/src/id.rs passes:
   two identifiers from the same millisecond share a prefix and differ in the
   tail (from crates/meow-core/src/id.rs:91-99, high).
2. `identifiers_sort_by_creation_time` in the same file passes (from
   crates/meow-core/src/id.rs:82-88, high).
3. `an_ambiguous_identifier_lists_the_candidates_rather_than_guessing` in
   crates/meow-session/tests/spec.rs passes (from
   crates/meow-session/tests/spec.rs:367-386, high).

## What this does not settle

- Where the entropy comes from, which ADR-0438 records (from `meow-method find
  short identifier`, high).
- The selectors `@last`, `@last-N` and `@<agent-name>` and session names, which
  [R-SESSION-032] and [R-SESSION-033] settle (from docs/spec/session.md,
  high).
