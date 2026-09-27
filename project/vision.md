---
id: vision
artifact: vision
status: live
revised: 2026-09-27
---

<!-- Written to the writing standard meow-prose ships: lead with the answer,
give each rule its reason in the same sentence, and show the failing case. -->

# meowg1k

## What it is

meowg1k is a runtime for AI commands you write yourself. The binary supplies the
parts that are hard to build and boring to rebuild: model providers, a session
log, budgets, a permission layer, retrieval over your code and a terminal that
behaves. You write the commands in Starlark, in a `.meow/` directory beside the
code they work on. (from the previous `docs/vision.md`, high)

It replaces two things. The first is an AI tool you operate through its own
interface, which doesn't give you a workflow you can diff, review, hand to a
colleague or put in front of `&&`. (from the previous `docs/vision.md`, high)
The second is meowg1k's own Go implementation, `v0.2.1`, whose workflows were
15,000 lines of Go that users couldn't change without rebuilding, whose guides
described modules that didn't exist, and which had no permission layer, so an
agent talked into running a command ran it. (from the previous `docs/vision.md`,
high)

This repository is the example. About 38,000 lines of Rust across ten crates
provide the runtime, and about 360 lines of Starlark and markdown in `.meow/`
give the project `meow review`, `meow commit`, `meow ask` and `meow status`.
Those 360 lines are all you read to know what the project's AI tooling does,
they go through code review, and changing one doesn't mean rebuilding a binary.
(from the previous `docs/vision.md`, high)

## The problem

You want `meow review` to gate a commit, and `meow ask` to answer from the code
instead of from a model's memory of code like it, and you want both to behave
the same on a laptop and in CI. (from the previous `docs/vision.md`, high) A
tool with its own interface keeps the workflow out of the repository, so it
can't be reviewed, versioned or shared with the code it belongs to. (from the
former `docs/philosophy.md` principle 6, high)

## Who it is for

| Audience | Wants | What they do today instead |
| -------- | ----- | -------------------------- |
| A developer who thinks in scripts | AI workflows that are code: a file to diff and review, a command to chain with `&&` (from the previous `docs/vision.md`, high) | Operates an AI tool through its own interface (from the previous `docs/vision.md`, high) |
| A team running the same workflow locally and in CI | Identical behaviour on a laptop and in CI (from the previous `docs/vision.md`, high) | Shell scripts that pipe text to a model's API and print the reply, which is the gap meowg1k's first issue set out to fill; the scripts carry no budget, no policy and no log (from https://github.com/retran/meowg1k/issues/1, medium) |

meowg1k asks more of someone happy with a tool's own interface than they need:
it asks them to write a handler and declare it. (from the previous
`docs/vision.md`, high)

## Quality goals

These are the principles the project has held since it began, from the former
`docs/philosophy.md`, which this section replaces. They're ranked here, and a
tie breaks toward the higher goal. The ranking was set during onboarding on
2026-09-27: security comes first because its failures outlive the run, and
configuration as code second because `CLAUDE.md` says the Starlark API wins when
it and the internals conflict. (reasoned from `CLAUDE.md`
`starlark_api_is_the_product`, low)

1. Security by design. meowg1k persists no secret of its own, no runtime module
   reaches the credential store, no command prints a credential, and releases
   carry a Sigstore attestation and an SPDX bill of materials. A `.meow/`
   directory in a freshly cloned repository is code with tool access, so the
   first run shows what it declares and asks once, and asks again when that
   changes. Partly met: credentials still live in a plaintext
   `~/.meow/auth.json`, which issue #166 tracks. (from the former
   `docs/philosophy.md` principle 9, high)
2. Configuration is code. `.meow/meow.star` is a Starlark program, so a workflow
   is versioned, reviewed and shared with its repository. Loading is exclusive:
   a project configuration means the global one isn't read, because a global
   setting bleeding into a project that didn't restate it makes the workspace
   behave differently on two machines. (from the former `docs/philosophy.md`
   principle 6, high)
3. Task execution, not a conversation. A run has a beginning, an end and a stop
   reason. The agent loop runs against a task under a budget and returns an
   outcome, and `--continue` resumes the most recent session of the invoked
   command or fails when it has none. (from the former `docs/philosophy.md`
   principle 2, high)
4. A composable engine. Every run exits with a code a shell can branch on, reads
   stdin, and can emit one JSON event per line with nothing else on stdout, so
   `meow review && git commit` works. A command a workspace declares is a
   top-level subcommand with its own `--help` and typed flags. (from the former
   `docs/philosophy.md` principle 1, high)
5. Predictable and auditable. The tool's own logic is deterministic, and
   randomness comes only from the model's sampling, which the user sets. Every
   turn, tool call, policy decision and token lands in an append-only log, and
   `--dry-run` decides policy and plans calls without making any. (from the
   former `docs/philosophy.md` principle 7, high)
6. Predictable cost. The user chooses the model, caps the tokens and sets a hard
   ceiling on usage, and a fan-out shares its caller's ledger. Partly met: the
   shared request rate limit is gone (issue #167), and the cost axis can't fire
   until models carry a price (issue #168). (from the former
   `docs/philosophy.md` principle 10, high)
7. Local-first. The index, the session log, the blobs and the response cache are
   one SQLite file in the workspace. Only the model needs a network, and a local
   server counts, so an indexed workspace searches offline. (from the former
   `docs/philosophy.md` principle 5, high)
8. No lock-in. Switching providers is a configuration change. Seven provider
   kinds ship: `anthropic`, `openai`, `openrouter`, `gemini`, `voyage`, `llama`
   and `copilot`, and every OpenAI-shaped vendor is served by one implementation
   that differs only in its address. (from crates/meow-cli/src/wire.rs `KINDS`,
   high)
9. Intelligent context. The workspace is walked under `.gitignore` and
   `.meowignore`, chunked on line boundaries, embedded and indexed into an HNSW
   graph, and `search.code` asks it by meaning. (from the former
   `docs/philosophy.md` principle 8, high)
10. One static binary with no runtime to install, which starts at once and runs
    from a laptop to a stripped container. (from the former `docs/philosophy.md`
    principle 3, high)
11. Radically open. Apache 2.0, developed in the open, with the specifications
    and the reasoning behind them in the repository. (from the former
    `docs/philosophy.md` principle 11, high)

A goal is an intention until a requirement enforces it. When no requirement
backs a goal, a disagreement about it is a design discussion; when one does and
the code fails it, that's a defect with an issue. (from the former
`docs/philosophy.md`, "What makes a principle binding", high)

| Goal | Enforced by |
| --- | --- |
| 1. Security by design | REQ-1200 to REQ-1212, REQ-1222 to REQ-1229, REQ-2524, REQ-2525 |
| 2. Configuration is code | REQ-2400 to REQ-2402, and SPC-2400 throughout |
| 3. Task execution | REQ-1001, REQ-2850, REQ-2851 |
| 4. A composable engine | REQ-2824 to REQ-2831, REQ-2852 to REQ-2854 |
| 5. Predictable and auditable | SPC-2200, REQ-2845 to REQ-2847 |
| 6. Predictable cost | REQ-1009 to REQ-1019 |
| 7. Local-first | SPC-2600, SPC-1400, REQ-1814, REQ-1815 |
| 8. No lock-in | SPC-1600, REQ-1600 to REQ-1606 |
| 9. Intelligent context | SPC-1400 |

## What it will not do

- Hold a conversation. A run is a task with an end and a stop reason, and
  `--continue` resumes the most recent session of the invoked command or fails
  when it has none. (from the previous `docs/vision.md`, high)
- Host plugins. A package is Starlark under the same rules as code you wrote,
  and nothing loads native code. (from the previous `docs/vision.md`, high)
- Tie you to a vendor. Every vendor that copied OpenAI's shape is served by one
  implementation that differs only in its address. (from the previous
  `docs/vision.md`, high)
- Run as a service. Nothing phones home. (from the previous `docs/vision.md`,
  high)
- Layer a global configuration under a project one, because a global setting
  bleeding into a project that didn't restate it makes the workspace behave
  differently on two machines. (from the former `docs/philosophy.md` principle
  6, high)

## Risks

- Credentials live in a plaintext `~/.meow/auth.json`, which breaks the security
  goal; issue #166 tracks moving them to the operating system's secret store.
  (from the former `docs/philosophy.md` principle 9, high)
- The rewrite dropped the shared request rate limit, `requestsPerMinute` and
  `requestsPerDay`; issue #167 tracks restoring it. (from the former
  `docs/philosophy.md` principle 10, high)
- A declared cost cap does nothing, because no provider reports a cost to the
  budget's `cost_micros` axis; issue #168 tracks it. (from the former
  `docs/philosophy.md` principle 10, high)
- A `.meow/` in a freshly cloned repository is code with tool access. The first
  run shows what it declares and asks once, and asks again when the declarations
  change. (from the previous `docs/vision.md`, high)

## Where it is going

The engine stays large so that the workspace stays small, and when the two
conflict, the workspace wins. (from the previous `docs/vision.md`, high) A
principle becomes binding when a requirement enforces it, so the direction is to
close the three partly enforced principles above with requirements and code.
(from the former `docs/philosophy.md`, "What makes a principle binding", high)
