---
id: ADR-2000
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2011, REQ-2012, REQ-2013, REQ-2014]
supersedes: []
---

# 2000. A multi-path read is partial and says so, and a multi-path write is all or nothing

## Decision

A read that hits a denied path returns the allowed paths and names every path it
skipped, and a write that hits a denied path is denied as a whole and writes no
path (from docs/spec/policy.md [R-POLICY-008] [R-POLICY-009], high).

Once this is accepted, `evaluate` returns the paths an allow rule's selector
missed in `Verdict::denied_paths`, and `Access` tells a read from a write (from
crates/meow-policy/src/policy.rs:64-76 and 146-147, high). What doesn't work
yet is acting on it: the engine runs any call whose decision is `allow` and
never reads `denied_paths`, so a read isn't narrowed and a write isn't refused
(from crates/meow-agent/src/engine.rs:357-399, high).

## Why

Naming the skipped paths removes the danger of a result that looks complete and
isn't, and denying the whole read would cost an agent its view of a repository
because of one `.env` file it never wanted (from docs/spec/policy.md, high). A
write is different, because a partial one leaves the workspace in a state nobody
chose (from docs/spec/policy.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Deny a multi-path read and a multi-path write alike when any path is denied | Never returns a result that looks complete and isn't (from docs/spec/policy.md, high) | Naming the skipped paths removes that danger, and denying the whole read costs an agent its view of a repository because of one `.env` file it never wanted (from docs/spec/policy.md, high) |
| Apply a write to the allowed paths only | The allowed part of the work gets done, the same way a partial read keeps the allowed part of the view (reasoned from docs/spec/policy.md, Decisions, low) | A partial write leaves the workspace in a state nobody chose (from docs/spec/policy.md, high) |
| Do nothing: judge the call once, not per path | One decision per call, with no per-path bookkeeping (reasoned from crates/meow-policy/src/policy.rs:300-316, low) | REQ-2010 requires one evaluation per resolved path, so a single decision can't express a partial result (from docs/spec/policy.md [R-POLICY-006], high) |

The first option was the spec's first answer, and a trade-off pass over the
requirements replaced it (from https://github.com/retran/meowg1k/pull/111,
high).

## What it costs

Every tool that touches several paths has to say whether it reads or writes,
and the runtime guesses it from the tool's name, treating `write`, `remove`,
`append` and `mkdir` as a write (from crates/meow-star/src/run.rs:525-535,
high). A read result has to carry the list of skipped paths back to the model,
which is a second output channel for each multi-path tool (reasoned from
crates/meow-policy/src/policy.rs:146-147, low).

## What would reverse it

- A model shown to treat a read that names its skipped paths as complete, so
  that naming them doesn't remove the danger the first answer feared, would
  reopen the read half (reasoned from docs/spec/policy.md, Decisions, low).

## Consequences

An agent keeps its view of a repository when one file in it is denied (from
docs/spec/policy.md, Decisions, high). The access kind of a tool becomes part
of what policy judges, so a tool whose name hides a write is judged as a read
(from crates/meow-star/src/run.rs:525-535, high).

## How I will know it was realised

1. `a_read_that_hits_a_denied_path_says_which_it_skipped` in
   crates/meow-policy/tests/spec.rs passes: the verdict is `allow` and names
   `/w/.env` (from crates/meow-policy/tests/spec.rs:152-174, high).
2. An engine test in crates/meow-agent/tests/spec.rs shows a multi-path read
   returning only the allowed paths and naming the rest, and a multi-path write
   touching no path. No such test exists:
   `a_write_that_hits_a_denied_path_writes_nothing` asserts only that
   `denied_paths` is non-empty and that the call is a write (from
   crates/meow-policy/tests/spec.rs:176-190, high).

## What this does not settle

- What happens when an explicit `deny` rule matches one path of a read: the
  deny rule matches the call and the whole read is denied, since only an allow
  rule's misses become `denied_paths` (from
  crates/meow-policy/src/policy.rs:255-283 and 300-316, high).
- How a tool declares whether it reads or writes, beyond the name convention
  (from crates/meow-star/src/run.rs:515-535, high).
