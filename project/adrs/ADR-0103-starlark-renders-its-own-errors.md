---
id: ADR-0103
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2526]
supersedes: []
---

# 0103. Starlark errors keep the diagnostics `starlark-rust` renders

## Decision

A Starlark error is shown as `starlark-rust` renders it, with a call stack and a
source span, and `miette` renders configuration and command-line errors, which
have no Starlark frame (from docs/design/0.3.0-spike-starlark.md Diagnostics
already carry spans, high).

What works today: `StarError::Starlark` carries the library's rendered text
unchanged (from crates/meow-star/src/error.rs:114-119 and
crates/meow-star/src/loader.rs:197, high). What doesn't: no crate depends on
`miette`, and `meow-cli` prints configuration and command-line errors with
`eprintln!("{error}")` (from `grep -rn miette crates`, no hits, and
crates/meow-cli/src/wire.rs:46, high).

## Why

The library already renders the file, line and column, and does it better than
the proposal, because it carries the call stack too (from
docs/design/0.3.0-spike-starlark.md Diagnostics already carry spans, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Render Starlark errors through `miette`, as the architecture document first proposed | One look for every error the binary prints, Starlark or not (reasoned from docs/design/0.3.0-architecture.md section 8, low) | Unnecessary: `starlark-rust` already does it, and carries the call stack as well (from docs/design/0.3.0-spike-starlark.md Diagnostics already carry spans, high) |
| Do nothing: format errors as strings, as v0.2.x did | No span bookkeeping in the runtime (reasoned from docs/design/0.3.0-architecture.md section 8, low) | Item 13 of the v0.2.x TODO asked for better error messages, and keeping spans in place of formatted strings is what makes them close to free (from docs/design/0.3.0-architecture.md section 8, high) |

A third option isn't recorded anywhere I searched: the design documents, the
code and the pull request history name only these two.

## What it costs

A configuration error and a Starlark error look different on screen, because
two renderers produce them (reasoned from docs/design/0.3.0-spike-starlark.md
Diagnostics already carry spans, low). The layout of a Starlark error belongs to
`starlark-rust`, so an upgrade of the library can change it (reasoned from
crates/meow-star/src/error.rs:114-119, low).

## What would reverse it

- A `starlark-rust` release that stops rendering the span or the call stack,
  which would remove the reason the spike gave (reasoned from
  docs/design/0.3.0-spike-starlark.md Diagnostics already carry spans, low).

## Consequences

The architecture document records the error handling as `thiserror` in libraries
and `miette` at the boundary for configuration and command-line errors (from
docs/design/0.3.0-architecture.md section 8, high). `meow-star` needs nothing
further for `[R-STAR-090]`, because the library's diagnostic already carries the
span (from crates/meow-star/src/error.rs:116-117, high).

## How I will know it was realised

1. `an_error_points_at_the_line_that_caused_it` in
   crates/meow-star/tests/loading.rs passes: a load error names `meow.star` and
   line 2 (from crates/meow-star/tests/loading.rs:386-394, high).
2. A failing `fs.read` in a script prints a traceback naming each frame and a
   span pointing at the call, as the spike showed (from
   docs/design/0.3.0-spike-starlark.md Diagnostics already carry spans, high).

## What this does not settle

- Whether `miette` is still wanted for configuration and command-line errors:
  the design says yes, CLAUDE.md says to convert to `miette` in `meow-cli` so a
  Starlark mistake renders with a pointer at the line, and the code uses
  neither (from docs/design/0.3.0-architecture.md section 8, CLAUDE.md:125-127
  and crates/meow-cli/src/wire.rs:46, high).
