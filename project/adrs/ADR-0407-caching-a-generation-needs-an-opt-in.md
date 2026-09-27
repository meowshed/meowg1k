---
id: ADR-0407
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2638, REQ-2639]
supersedes: []
---

# 0407. Caching a generation needs the caller to name it and opt in

## Decision

The cache makes `R-STORE-045` structural rather than a rule to remember: no way
exists to cache a generation without naming `CacheKind::Generation` and passing
an opt-in (from https://github.com/retran/meowg1k/pull/113, high).
`Store::cache_put` takes a `CacheKind` and an `opted_in` flag, and a
generation written with `opted_in` false is a no-op that returns `false`, never
an error, so a caller that forwards a flag doesn't have to branch (from
crates/meow-store/src/cache.rs:47-62, high).

Once this is accepted, embeddings are cached by default and the index's query
path uses the cache (from crates/meow-index/src/index.rs:434-466, high). No
caller caches a generation today: the index is the only caller of
`cache_put`, and nothing in the Starlark surface or the command line offers the
opt-in, so a user can't yet ask for a cached generation (from `grep -rn
cache_put crates`, which finds only crates/meow-index/src/index.rs:456 and the
store's tests, high).

## Why

With the opt-in in the type, the default path can't cache a generation by
accident (from https://github.com/retran/meowg1k/pull/113, high). An agent that
retries wants a fresh attempt, and a cache would hand it the answer that
already failed (from docs/spec/store.md [R-STORE-045], high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: keep v0.2.x's caching decorator, which wrapped every generation call when a preset set `CacheEnabled` and `--no-cache` was absent | One switch per preset turned caching on for every call without touching code (from v0.2.1:internal/adapters/gateway/factory.go:58-78, high) | The switch covered every call made with that preset, retries included, so a retry could get back the answer that had failed; the spec forbids caching a generation unless the caller asks (reasoned from docs/spec/store.md [R-STORE-045] and v0.2.1:internal/adapters/gateway/caching.go, low) |
| A rule callers remember, with one untyped `cache_put` | A smaller signature, with no kind and no flag at each call site (reasoned from crates/meow-store/src/cache.rs:53-60, low) | The default path could cache a generation by accident (from https://github.com/retran/meowg1k/pull/113, high) |
| Name the kind and the opt-in in the call (chosen) | The compiler refuses a generation cache write that doesn't say `CacheKind::Generation` (from crates/meow-store/src/cache.rs:16-27, high) | It won |

## What it costs

Every call site passes two more arguments, including the index, which has to
pass `CacheKind::Embedding` and a flag the embedding path ignores (from
crates/meow-index/src/index.rs:456-463, high). A generation write with the flag
false is dropped silently and reported only through the returned `bool`, so a
caller that ignores the result can't tell its entry wasn't stored (from
crates/meow-store/src/cache.rs:48-51, high). `cache_get` takes no kind, so the
guard holds only on the write side (from crates/meow-store/src/cache.rs:36-45,
high).

## What would reverse it

- `R-STORE-045` is amended to let generations be cached by default, which
  removes the obligation this type exists to enforce (reasoned from
  crates/meow-store/src/cache.rs:16-20, low).
- Every generation call site passes `opted_in` as a constant `true`, which
  shows the flag no longer carries a choice (reasoned from
  crates/meow-store/src/cache.rs:53-60, low).

## Consequences

- `CacheKind` is part of the store's public interface, re-exported from
  `meow_store` (from crates/meow-store/src/lib.rs:29, high).
- A future generation cache has to reach the store through a caller that
  decides the opt-in, so the Starlark surface or a model declaration needs an
  explicit setting before one exists (reasoned from
  crates/meow-store/src/cache.rs:47-62, low).
- The model is part of the key, so an opted-in generation from one model is
  never returned for another (from crates/meow-store/src/cache.rs:31-34, high).

## How I will know it was realised

1. `generations_are_not_cached_unless_the_caller_asks` in
   crates/meow-store/tests/spec.rs passes: an embedding written with the flag
   false is read back, a generation written with the flag false misses, and the
   same generation written with the flag true is read back (from
   crates/meow-store/tests/spec.rs:382-418, high).

## What this does not settle

- Where a user or a script says it wants a generation cached: no Starlark
  argument, model setting or flag exists for it yet (from `grep -rn cache
  docs/design/0.3.0-starlark-api.md docs/spec/starlark.md`, which finds no such
  setting, high).
- How the request hash for a generation is computed; the index hashes the
  query text alone, which suits an embedding and not a request with messages
  and tools (from crates/meow-index/src/index.rs:436, high).
