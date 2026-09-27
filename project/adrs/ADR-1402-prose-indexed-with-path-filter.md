---
id: ADR-1402
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-1408, REQ-1433, REQ-1434]
supersedes: []
---

# 1402. Prose is indexed alongside code, and a query narrows with a path filter

## Decision

Indexing doesn't exclude prose, and a query narrows its results with a path
filter (from docs/spec/index.md [R-INDEX-005] [R-INDEX-044], high).

Once this is accepted, Markdown and plain text go through the same walk, the
same exclusions and the same chunker as code, because the walk has no rule
about prose at all (from crates/meow-index/src/walk.rs:81, high). A caller
narrows a query with globs from `search.code(paths=[...])` in Starlark or
`meow index query --path` on the command line, and the filter decides which
graph nodes the walk may visit (from crates/meow-star/src/modules.rs:793-811,
crates/meow-cli/src/surface.rs:284-288 and
crates/meow-index/src/index.rs:324-329, high). What still doesn't work is any
default narrowing: a query with no filter ranks prose and code together. No
command in this repository's `.meow/` calls `search.code` yet, so the filter is
untried on the workspace meowg1k uses on itself (from
`grep -rn search.code .meow`, which finds no call, high).

## Why

Excluding prose would make the design documents unsearchable by the agents most
likely to need them. Dilution is the caller's to solve with a filter, and the
index doesn't solve it by guessing (from docs/spec/index.md, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: index prose as v0.2.x did and add no path filter | Nothing to build: v0.2.x already chunked `.md` and `.txt` with its plain-text strategy (from `git show v0.2.1:internal/core/chunker/service.go`, high) | A caller has no way to narrow the results, so prose dilutes code results with nothing to push back (reasoned from docs/spec/index.md, low) |
| Exclude prose from the index | Less dilution of code results (from docs/spec/index.md, medium) | The design documents become unsearchable by the agents most likely to need them (from docs/spec/index.md, high) |
| Let the index weight or separate prose by itself | The caller needs to pass nothing (reasoned from docs/spec/index.md, low) | The index would be guessing what a caller wants, and dilution is the caller's to solve with a filter (from docs/spec/index.md, high) |
| Apply the path filter to the results after ranking | Simpler: the graph walk stays unfiltered (reasoned from crates/meow-index/src/index.rs:324-329, low) | It returns the best ten overall and then throws most of them away, so a filtered query can come back empty (from crates/meow-index/src/index.rs:324-329 and https://github.com/meowshed/meowg1k/pull/131, high) |

## What it costs

Results can be diluted by prose, and the caller narrows them with a path filter
(from docs/spec/index.md, high). A filtered query first matches the path of
every indexed chunk against the globs to build the list of nodes the walk may
visit (from crates/meow-index/src/index.rs:413-432, high). The path filter
adds `globset` to `meow-index`, which was already in the tree through
`meow-policy` (from https://github.com/meowshed/meowg1k/pull/131, high).

## What would reverse it

- Prose that the index returns ahead of code, on queries about code, measured
  often enough that callers pass a filter on most queries (reasoned from
  docs/spec/index.md, low).

## Consequences

The walk carries no rule about file types, so prose and code follow the same
`.gitignore` and `.meowignore` exclusions (from
crates/meow-index/src/walk.rs:81, high). `search.code` takes a `paths` list and
`meow index query` takes a repeatable `--path`, and both pass the globs to the
same query (from crates/meow-star/src/modules.rs:793-811 and
crates/meow-cli/src/surface.rs:284-288, high).

## How I will know it was realised

1. `prose_is_indexed_alongside_code` in crates/meow-index/tests/walk.rs passes:
   `README.md`, `docs/design.md` and `notes.txt` are indexed beside
   `src/main.rs`.
2. `a_path_filter_applies_before_ranking` in crates/meow-index/tests/index.rs
   passes: with a limit of one and a `docs/**` filter, the query returns the
   `docs/` chunk although the best match overall is in `src/`.

## What this does not settle

- Whether a workspace should exclude its own prose. `.meowignore` still can,
  under [R-INDEX-001] (from docs/spec/index.md, medium).
- The glob syntax. It is `globset`'s, shared with `meow-policy` (from
  https://github.com/meowshed/meowg1k/pull/131, high).
