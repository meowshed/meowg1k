---
id: ADR-2400
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2524, REQ-2525]
supersedes: []
---

# 2400. Declaration files may read the environment and nothing else

## Decision

While `.meow/` is being evaluated, `load` and `@std//env` are callable and every
other runtime module fails (from docs/spec/starlark.md [R-STAR-084], high).

## Why

Credentials are resolved in the environment, so forbidding it outright would
make the normal configuration impossible. Every other module stays unavailable,
so loading `.meow/` can't have consequences (from docs/spec/starlark.md,
Decisions, high).

## Alternatives

| Option | Better at | Why it lost |
| ------ | --------- | ----------- |
| Forbid every runtime module during declaration, `@std//env` included | Loading `.meow/` would read nothing from the machine, so a declaration would evaluate the same everywhere (reasoned from docs/spec/starlark.md [R-STAR-083], low). | Credentials are resolved in the environment, so forbidding it would make the normal configuration impossible (from docs/spec/starlark.md, Decisions, high). |
| Do nothing: keep [R-STAR-083] alone, which forbade side effects during declaration with no carve-out | No second rule to keep consistent with the first (reasoned from PR #110, low). | The design document's own `env.require` example broke it, which is one of the ten defects PR #110 found (from https://github.com/retran/meowg1k/pull/110, high). |
| Let declaration call every module, as a handler can | A declaration could compute what it declares from files or commands (reasoned from docs/spec/starlark.md [R-STAR-083], low). | [R-STAR-083] forbids a declaration file to write files, run commands or make network requests, and the trust prompt is safe only because declaring reaches nothing (from docs/spec/starlark.md [R-STAR-083] and https://github.com/retran/meowg1k/pull/154, high). |

## What it costs

A declaration can depend on the machine it loads on, because `env.get` returns
whatever the environment holds (reasoned from crates/meow-star/src/modules.rs
module header, low). Each runtime builtin pays one check: it calls `running`
first, which refuses it during declaration (from
crates/meow-star/src/run.rs `running`, high).

## What would reverse it

The carve-out loses its reason if a declaration can no longer name a
credential: `api_key` is the first place [R-AUTH-001] looks, and the design
calls `api_key = env.require(...)` the escape hatch beside the global store and
the provider's environment variable (from docs/spec/auth.md [R-AUTH-001] and
docs/design/0.3.0-starlark-api.md section 4.1, high). If `api_key` left
`meow.provider`, forbidding `@std//env` too would cost nothing (reasoned from
crates/meow-cli/src/wire.rs `credential`, low).

## Consequences

Every runtime module resolves in both phases, and its builtins fail during
declaration with `` `<module>.<call>` is not available while .meow/ is being
loaded `` (from crates/meow-star/src/modules.rs module header and
crates/meow-star/src/error.rs `ModuleUnavailable`, high). `meow trust` and
`meow pkg` load a workspace before anyone agreed to run it, and that is safe
because loading reaches no file, program or network (from
https://github.com/retran/meowg1k/pull/154 and
https://github.com/retran/meowg1k/pull/159, high). This repository's own
`.meow/meow.star` reads its keys with `get` from `@std//env` (from
.meow/meow.star, high).

## How I will know it was realised

`only_env_is_callable_during_declaration` in
crates/meow-star/tests/loading.rs loads `@std//env` and calls it, and calls
`text.tokens` and gets the refusal.
`fs_and_shell_refuse_to_run_during_declaration`,
`re_and_time_are_refused_during_declaration`,
`the_encoders_are_refused_during_declaration`,
`the_store_is_refused_during_declaration`, `http_is_refused_during_declaration`
and `search_and_index_are_refused_during_declaration` in
crates/meow-star/tests/running.rs do the same for each other module (from those
tests, high).

## What this does not settle

Which environment variables a declaration may read: `@std//env` exposes `get`
and `require` with no allow-list, so any variable is readable (from
docs/design/0.3.0-starlark-api.md section 10 and
crates/meow-star/src/modules.rs `env_module`, high).
