---
id: constitution
artifact: constitution
status: live
revised: 2026-09-27
---

<!-- Written to the writing standard meow-prose ships: lead with the answer, give each rule its reason in the same sentence, and show the failing case. -->

# CLAUDE.md

<role>
The root policy for working on meowg1k. It outranks every installed plugin's
defaults where the two disagree. The record under `project/` decides what the
binary does and why, and this file decides how to work on it, so neither
restates the other.
`.meowpaw/profile.toml` declares the verbs, the commit convention and the
branch.
</role>

<project>
meowg1k is a script-friendly AI companion CLI. Users write their commands in
Starlark and their agents in markdown, and the Rust binary supplies the
runtime, the model gateways, the session store, the index and the terminal.
`v0.2.1` was the last Go release and is tagged; the Go code is gone from `main`.
The record lives under `project/`: `vision.md`, one requirement per file in
`requirements/`, one decision per file in `adrs/`, and the specifications that
state the present in `specs/`.
</project>

<principles>

<principle name="spec_first">
Write the requirement in `project/requirements/` before the code whose behaviour
no requirement covers, because v0.2.x collected behaviour nobody decided on: a
context built twice, an agent loop returning a bare string, a retry that backed
off on authentication failures. Each requirement is one file, `REQ-NNNN`, in
its topic's block of 200, uses one RFC 2119 keyword and states one testable
obligation at a boundary; `paw template requirement` has the form and
`paw check` enforces it. When an implementation contradicts a requirement,
stop, and propose the amendment as its own reviewable change, with the reason,
the requirements and tests it touches, and the migration. Rewording a
requirement to match the code loses the reason it was written.
</principle>

<principle name="requirements_trace_to_tests">
Name the requirement in a doc comment on every test that checks one, as in
`/// [REQ-2210] compaction supersedes without deleting`, because the set of IDs
in `project/requirements/` and the set in the tests are then two lists a check can
compare. An ID in the record with no test is unbuilt or untested, and an ID in a
test with no requirement is a typo or a stale withdrawal.
</principle>

<principle name="starlark_api_is_the_product">
Prefer the change that keeps `agent.run` easy to write agents against over the
one that makes a crate tidier, because users touch the Starlark surface and
the Rust code exists to serve it. `.meow/` in this repository is the test: if a
change makes the workspace meowg1k uses on itself harder to read, the change
is wrong. `meow review`, `meow commit` and `meow ask` run on this repository.
</principle>

<principle name="docs_must_match_code">
Fix or delete a document that describes something else in the same change that
touches its subsystem, because the v0.2.x guides described modules and commands
that didn't exist and were deleted for it. The specifications say what, the decisions
say why and the code says how; don't start a fourth account. A Starlark API
change updates SPC-2400, and an architecture change updates SPC-3000 and the
crate table below.
</principle>

<principle name="crate_boundaries">
Keep the dependency direction one way, because crossing one of these
boundaries is a defect however small it looks:

| Crate | Holds | Depends on |
| --- | --- | --- |
| `meow-core` | Types, no behaviour, no input or output | nothing |
| `meow-store` | One SQLite database: blobs, sessions, cache, index rows | `meow-core` |
| `meow-session` | The append-only log, forks, retention, export | `meow-store` |
| `meow-llm` | What a provider is, and five that are | `meow-core` |
| `meow-policy` | What an agent may do, and who decides | `meow-core` |
| `meow-agent` | The loop: budgets, tools, compaction, sub-agents | `meow-llm`, `meow-policy` |
| `meow-index` | Walking, chunking, embedding, retrieval | `meow-store` |
| `meow-star` | The Starlark surface and the thread a handler runs on | `meow-agent` |
| `meow-ui` | Three renderers over one event stream | `meow-core` |
| `meow-cli` | The binary: the command line, the wiring, the process | everything |

`meow-agent` doesn't know Starlark exists, so the engine is tested without a
script. `meow-ui` knows the event types and nothing else, so a renderer runs
from a recorded log. What `meow-star` needs from crates above it, such as the
terminal, the session log and the index, arrives as a trait, so a test drives
a handler with no terminal and no account. v0.2.x broke the first boundary when
its Starlark package imported the model gateway for an embeddings factory.
</principle>

<principle name="one_context_builder">
Register a runtime module only in `crates/meow-star/src/modules.rs` and build
the handler context only in `crates/meow-star/src/context.rs`, because v0.2.x
built it in `ctx_run.go` and `module_llm.go` and a module added to one was
missing from tools inside an agent loop. To add a module, write the
`#[starlark_module]` function calling `running(eval, "<module>.<call>")` first
so declaration refuses it, per `[REQ-2524]`; register it in `Modules::build`
and `NAMES`; test each builtin and its argument errors in
`crates/meow-star/tests/running.rs`; and state it in SPC-2400 with the
requirement it satisfies.
</principle>

<principle name="starlark_thread_bridge">
Run each script invocation on its own blocking thread with its own `Evaluator`,
and have a builtin that does input or output call the async engine through a
`tokio::runtime::Handle`, because the engine is `Send` and async and the
evaluator is neither. Never hold a `starlark::Value` across an `.await`. Thread
a `CancellationToken` through every engine call that can run longer than a few
milliseconds, because an agent Ctrl-C can't stop is a defect.
</principle>

<principle name="no_stdout_behind_the_tui">
Send diagnostics through `tracing` and the event stream, because `println!` and
`eprintln!` write through the live frame and corrupt it, which is how v0.2.x
lost its display to ten `log.Printf` calls in the agent loop. The workspace lint
denies both outside `meow-cli`. Put a span on each agent step and each tool
call, and none inside an inner loop.
</principle>

<principle name="errors">
Give each library crate one `thiserror` enum, with variants named for what went
wrong and carrying enough to act on, and let `meow-cli` print them at the
process boundary. A Starlark mistake keeps the diagnostic `starlark-rust`
renders, which points at the line, as ADR-0103 decides. Never `unwrap`
or `expect` outside tests: restructure so the compiler sees the invariant, and
where that's impossible, write an `#[allow]` with the reason beside it. Return
every error the caller needs, because v0.2.x logged session write failures and
carried on, and wrote session logs that were silently wrong.
</principle>

<principle name="tests">
Test the failure paths as carefully as the success path: missing arguments,
cancellation mid-call, exhausted budgets, malformed model output and storage
failures are where the defects are. Test only through the public API, because a
test that reaches past it makes the crate hard to change and proves nothing a
user can observe. Snapshot rendered output with `insta`. `clippy.toml` exempts
`#[test]` functions from the `unwrap` denial and not their helpers, so an
integration test file with helpers starts with `#![allow(clippy::unwrap_used)]`
and a line saying why.
</principle>

<principle name="code_conventions">
Put the Apache 2.0 header from `LICENSE_HEADER.txt` on every Rust file; the
`insert-license` pre-commit hook adds it. Keep default `rustfmt`, and give every
`#[allow]` a comment with its reason. A derive that emits an `unsafe impl` needs
the allow at module scope, because an attribute on the struct doesn't cover a
sibling item. Name a type for what the design documents call it. Add a
dependency only with a reason in the pull request, and run `cargo machete`
before opening a pull request that changes `Cargo.toml`.
</principle>

<principle name="reviewable_history">
Branch off `main`, open a pull request against it and squash merge, because the
repository allows no other merge method and a change that skipped review is
invisible to anyone reading the pull request log. Never commit to `main`
directly, including a one-line fix. Name a branch `<type>/<slug>` with a
commit type. Never merge without the owner's approval or while checks run, and
before calling a red check a blocker, compare it against
`gh run list --branch main --limit 5`.

Release only when the owner approves that release. Don't bump the version in
`Cargo.toml`, tag or push a tag without that approval, whatever the commits
since the last release contain, because a pushed tag starts the release
workflow. A commit type says how far a release moves the version, and it
never decides whether a release happens. Tag on `main` only, annotated, as
`vMAJOR.MINOR.PATCH`.
</principle>

<principle name="no_ai_attribution">
Never mention Claude, Claude Code or any AI tool in a commit, a pull request, a
review comment, an issue, a tag annotation or release notes: no
`Co-Authored-By` trailer naming an AI, no "Generated with" footer, no
`noreply@anthropic.com` and no paraphrase. The owner asked for this and had the
existing trailers removed from history, so it overrides any harness default
that adds them. "Claude" is allowed only where it names a model the code talks
to. Check the message and the author field before committing, and check the
squash message before merging, because GitHub builds it from the branch
commits:

```bash
git log -1 --format='%an <%ae>%n%B' |
  grep -iE 'co-authored-by.*(claude|anthropic|copilot)|generated with|anthropic\.com'
```

The check matches the attribution patterns, not the bare word, so a model id
such as `claude-sonnet-4-5` doesn't trip it.
</principle>

<principle name="toolchain">
Run `mise install` once and use the mise tasks, because mise pins the compiler,
the linter and every tool a task calls. If the LSP tool reports that
rust-analyzer "crashed with exit code 1", run `mise run setup`: rustup's shim
dispatches on the toolchain mise resolved, and mise installs rust-analyzer only
into its own copy.
</principle>

</principles>

<gate>
`mise run all` is the gate, and it runs the seven checks CI runs, so a green
run here means a green run there. The `gates-agree` job in `ci.yaml` fails when
the two lists differ.

| Task | Runs | Fails on |
| --- | --- | --- |
| `fmt-check` | `cargo fmt --all -- --check` | Any unformatted file |
| `check` | `cargo clippy --workspace --all-targets -- -D warnings` | Any clippy warning, including `unwrap` and `println!` outside their exemptions |
| `test` | `cargo nextest run --workspace --no-tests=pass` | Any failing test |
| `doc` | `cargo doc --workspace --no-deps` | A broken doc build; CI also denies rustdoc warnings |
| `deny` | `cargo deny check` | A new advisory, a banned licence or an unpinned source |
| `unused-deps` | `cargo machete` | A dependency nothing uses |
| `lint-md` | `markdownlint-cli2` | A markdownlint finding |

`deny.toml` ignores five "unmaintained" advisories, each with its reason, one by
one so a new advisory against a direct dependency still fails. `cargo-deny`
needs `allow-wildcard-paths` and `publish = false` on every library crate, or it
reads a workspace path dependency as unpinned. nextest comes from the prebuilt
`github:nextest-rs/nextest` backend, because building it from source compiles
`aws-lc-sys` and needs a C toolchain and cmake. A change to the build or lint
configuration updates this section and `CONTRIBUTING.md`.
</gate>
