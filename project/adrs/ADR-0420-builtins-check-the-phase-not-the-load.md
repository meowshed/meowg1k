---
id: ADR-0420
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2524, REQ-2525, REQ-2416]
supersedes: []
---

# 0420. Every module loads in both phases, and each builtin refuses a call made during declaration

## Decision

A declaration file may now load a runtime module and is refused when it calls
one: every module resolves in both phases and each builtin checks the phase
(from https://github.com/retran/meowg1k/pull/123, high). The check is
`running(eval, "<module>.<call>")`, which returns the run state or fails with
`StarError::ModuleUnavailable` during declaration (from
crates/meow-star/src/run.rs:950-970, high).

Once this is accepted, `load("@std//path", "join")` can sit at the top of a
file that also declares a tool, and calling any runtime builtin outside `env`
while `.meow/` is evaluated fails with "not available while .meow/ is being
loaded" (from crates/meow-star/tests/loading.rs
`one_table_separates_an_unknown_module_from_an_unavailable_call`, high). What
still depends on each author is the check itself: a builtin that doesn't call
`running` first isn't refused, and `CLAUDE.md` makes that call step one of
adding a module (from CLAUDE.md `<starlark_runtime>`, high).

## Why

`R-STAR-084` says only `load` and `@std//env` are callable during declaration
(from https://github.com/retran/meowg1k/pull/123, high). Refusing the load
itself made `load("@std//path", "join")` at the top of a file that also declares
a tool impossible (from https://github.com/retran/meowg1k/pull/123, high).
Checking in each builtin is also what makes `R-STAR-011` hold without a
mechanism of its own (from https://github.com/retran/meowg1k/pull/123, high):
both phases resolve a module through the one function `std_module`, so a
module can't exist in one phase and not the other (from
crates/meow-star/src/loader.rs:427-443, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Refuse the `load` of a runtime module during declaration, as the first implementation in #121 did | One check at the loader covers every module, so a new builtin can't forget it (reasoned from crates/meow-star/src/loader.rs:427-443, low) | A file that declares a tool couldn't load a module at its top for a handler to use (from https://github.com/retran/meowg1k/pull/123, high) |
| Two module tables, one per phase, with the declaration table holding only `env` | The declaration phase can't reach a runtime builtin at all, because none is in its table (reasoned from crates/meow-star/src/modules.rs:1-17, low) | A module could then exist in one phase and not the other, which is the drift REQ-2416 and the one-table rule exist to prevent (from crates/meow-star/src/modules.rs:5-12 and CLAUDE.md `one_context_builder`, high) |

Doing nothing was the first alternative: it was the code that had merged in
pull request 121 (from https://github.com/retran/meowg1k/pull/123, high).

## What it costs

Every runtime builtin has to call `running` before it acts, which is 54 call
sites across `modules.rs`, `capability.rs`, `capability_http.rs` and
`value.rs` (from `grep -c "running(eval"` over crates/meow-star/src, high).
Each module needs its own refusal test, and pull request 123 changed behaviour
that had merged in pull request 121 (from
https://github.com/retran/meowg1k/pull/123 and
crates/meow-star/tests/running.rs, high). `@std//git` doesn't call `running`
itself: it reaches the check through `run_words`, after checking its arguments,
so a declaration-time `git.diff(revision = "-x")` fails on the argument rather
than on the phase (from crates/meow-star/src/capability_git.rs:25-80 and
crates/meow-star/src/capability.rs:296-302, high).

## What would reverse it

- A builtin is found that acts during declaration because it didn't call
  `running`, and a per-builtin check can't be made mechanical; a check at the
  loader or a second table would then be worth its cost (reasoned from
  crates/meow-star/src/run.rs:950-970, low).

## Consequences

- `modules.rs` states that every module resolves in both phases and that only
  `env` answers during declaration (from crates/meow-star/src/modules.rs:14-17,
  high).
- Trust can be asked after loading, because loading reaches nothing (from
  crates/meow-cli/src/trust.rs:10 and crates/meow-cli/src/lib.rs:95, high).
- A declaration phase call and a run phase call get different messages: "not
  available while .meow/ is being loaded" in the first and "is not available
  here" with no run state at all (from crates/meow-star/src/run.rs:958-968,
  high).

## How I will know it was realised

1. `only_env_is_callable_during_declaration` and
   `one_table_separates_an_unknown_module_from_an_unavailable_call` in
   crates/meow-star/tests/loading.rs pass.
2. The per-module refusal tests in crates/meow-star/tests/running.rs pass:
   `fs_and_shell_refuse_to_run_during_declaration`,
   `re_and_time_are_refused_during_declaration`,
   `the_encoders_are_refused_during_declaration`,
   `the_store_is_refused_during_declaration`,
   `http_is_refused_during_declaration` and
   `search_and_index_are_refused_during_declaration`.
3. `a_tool_in_an_agent_loop_sees_the_same_modules` in the same file passes for
   REQ-2416.

## What this does not settle

- `@std//git` has no refusal test during declaration (from
  `grep -n "during_declaration" crates/meow-star/tests/*.rs`, high).
- Whether a builtin that checks its arguments before the phase, as
  `@std//git` does, meets REQ-2525's "fail with an error saying it is
  unavailable during declaration" (from
  crates/meow-star/src/capability_git.rs:25-80, high).
