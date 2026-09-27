---
id: ADR-0417
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2501]
supersedes: []
---

# 0417. A `.md` prompt file loads as a module with one symbol

## Decision

A `.md` prompt file loads as a module with one symbol rather than through a
builtin of its own (from https://github.com/retran/meowg1k/pull/122, high). The
symbol is `text`, bound to the file's contents, so a declaration writes
`load("//lib/style.md", "text")` (from crates/meow-star/src/loader.rs:288-298
and crates/meow-star/tests/agents.rs `a_prompt_file_loads_as_a_string`, high).

Once this is accepted, any `.md` file a `//` load names becomes a string in the
loading file, through the same path check and cache as a `.star` file (from
crates/meow-star/src/loader.rs:216-231, high). Nothing composes the string for
the user: joining it with other text is ordinary Starlark (from
crates/meow-star/tests/agents.rs `a_prompt_file_loads_as_a_string`, high).

## Why

The escape check, the cycle check and the evaluate-once cache then apply to it
without being written a second time (from
https://github.com/retran/meowg1k/pull/122, high). A markdown file has no
Starlark in it, so it becomes a module with one symbol rather than something to
evaluate (from crates/meow-star/src/loader.rs:220-224, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| A builtin of its own for prompt files | A call such as `meow.prompt("lib/style.md")` returns the string directly, with no symbol name to learn (reasoned from crates/meow-star/src/loader.rs:288-298, low) | The escape check, the cycle check and the evaluate-once cache would be written a second time (from https://github.com/retran/meowg1k/pull/122, high) |

Doing nothing was not an option, because REQ-2501 requires a `.md` file under
`.meow/lib/` to be loadable as a string (from docs/spec/starlark.md
[R-STAR-053], high). No third option appears in the code, the history or the
design documents.

## What it costs

A user has to know that the symbol is called `text`, and binds it to another
name with `load("//lib/style.md", style = "text")` when two prompt files are
loaded into one file (reasoned from crates/meow-star/src/loader.rs:291-293,
low). The `include` key in a markdown agent reads the same files through a
second function, `prompt_text`, which trims the contents, while the `load` path
keeps them as written, so one file can give two slightly different strings
(from crates/meow-star/src/loader.rs:300-312 and 288-298, high).

## What would reverse it

- A prompt file gains something to evaluate, such as substitution or a
  conditional, so it no longer maps onto one fixed string. REQ-2505 forbids
  that for markdown agents today (from
  docs/requirements/REQ-2505-markdown-agent-no-templating.md, high).

## Consequences

- `Loader::local` sends a path ending in `.md` to `prompt_module` before the
  cycle check, after the path has passed `resolve_local` (from
  crates/meow-star/src/loader.rs:216-231, high).
- The file is added to the load order, so it counts among the files a load
  reports (from crates/meow-star/src/loader.rs:290, high).

## How I will know it was realised

1. `a_prompt_file_loads_as_a_string` in crates/meow-star/tests/agents.rs
   passes: `load("//lib/style.md", "text")` gives the file's contents, and an
   agent's `system` built from it reads as expected.

## What this does not settle

- Whether the cycle check really applies: the `.md` branch returns before the
  cycle check runs (from crates/meow-star/src/loader.rs:225-240, high). That is
  harmless only because a markdown file loads nothing and so can't close a
  cycle (reasoned from the same lines, low).
- Whether a `load` may name a `.md` file outside `.meow/lib/`: `local` accepts
  any `.md` path that passes `resolve_local`, and REQ-2501 names only
  `.meow/lib/` (from crates/meow-star/src/loader.rs:216-231, high).
- Whether `load` and `include` should give the same string for one file (from
  crates/meow-star/src/loader.rs:300-312, high).
