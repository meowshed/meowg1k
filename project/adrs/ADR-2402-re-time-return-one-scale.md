---
id: ADR-2402
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2421, REQ-2422, REQ-2423, REQ-2424, REQ-2425, REQ-2426]
supersedes: []
---

# 2402. `re` and `time` return one scale each

## Decision

A `re` match is a list of groups or `None`, and a `time` instant is seconds in
UTC (from docs/spec/starlark.md, Decisions [R-STAR-013] [R-STAR-014], high).

## Why

A regular expression module that sometimes returns a string and sometimes a list
forces every caller to test the type first. A time module that knows about zones
turns every comparison into a question about where the machine is (from
docs/spec/starlark.md, Decisions, high). What handlers measure is durations
(from https://github.com/meowshed/meowg1k/pull/140, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| A `re` module whose result is sometimes a string and sometimes a list | A pattern with no groups gives the matched text directly, with no `[0]` (reasoned from crates/meow-star/src/modules.rs `re_module`, low). | Every caller has to test the type first (from docs/spec/starlark.md, Decisions, high). |
| A `time` module that knows about zones | A handler could print local wall-clock time for the person at the terminal (reasoned from crates/meow-star/src/modules.rs `time_module`, low). | Every comparison becomes a question about where the machine is (from docs/spec/starlark.md, Decisions, high). |
| Do nothing: no `re` or `time` module, as before PR #140 | No new surface and no new direct dependencies (reasoned from https://github.com/meowshed/meowg1k/pull/140, low). | The design table had promised fifteen modules and had eight, and these two need no port, so they came first (from https://github.com/meowshed/meowg1k/pull/140, high). |

## What it costs

A handler that wants the matched text writes `m[0]` (from
crates/meow-star/src/modules.rs `re.match`, high). Every call compiles its
pattern, which costs microseconds, and a handler matching in a hot loop over a
large corpus pays it each time (from https://github.com/meowshed/meowg1k/pull/140,
high). A handler can't show a person local time: `time.format` rejects `%Z`
with "requires IANA time zone identifier" and prints `%z` as `+0000` whatever
`TZ` is (from a probe of `target/debug/meow` built on 2026-09-27 with
`TZ=Asia/Tokyo`, high).

## What would reverse it

A requirement that a handler print local time for a person would reopen the
`time` half, because `time.format` can't produce it today (reasoned from the
probe above, low). A measured cost of compiling a pattern per call would add a
cache keyed on the pattern, and that doesn't change the surface (from
https://github.com/meowshed/meowg1k/pull/140, high).

## Consequences

`re.match` returns `None` for no match and a list for a match, and inside the
list a group that took no part is `None`, which is how a handler tells "matched
empty" from "did not match" (from https://github.com/meowshed/meowg1k/pull/140,
high). `re.find_all` takes a `limit`, because a pattern that can match empty is
unbounded on a long subject (from https://github.com/meowshed/meowg1k/pull/140,
high). `time.parse` of `2026-01-01T00:00:00+02:00` returns the UTC instant
`1767218400` (from the probe above, high). Formatting for a human is the job of
`time.format`, the only place a zone could ever enter (from
docs/spec/starlark.md, Decisions, high).

## How I will know it was realised

In crates/meow-star/tests/running.rs,
`no_match_is_none_and_an_empty_match_is_a_list`,
`find_all_replace_and_split_agree_about_what_matched`,
`find_all_stops_at_the_limit_it_was_given` and
`a_pattern_that_does_not_compile_names_itself_and_the_reason` check `re`, and
`parse_and_format_round_trip_in_utc`,
`since_is_negative_for_an_instant_that_has_not_happened` and
`text_that_is_not_a_timestamp_names_itself` check `time` (from those tests,
high). No test runs `time` under a non-UTC `TZ`, which is what [R-STAR-014]
promises (reasoned from crates/meow-star/tests/running.rs, medium).

## What this does not settle

The layout vocabulary of `time.format`: it passes `layout` to `jiff`'s
`strftime`, so the accepted fields are `jiff`'s and [R-STAR-014] fixes only the
scale and the default (from https://github.com/meowshed/meowg1k/pull/140, high).
Whether `time.format` should ever take a zone: today it takes none (from
crates/meow-star/src/modules.rs `time.format`, high).
