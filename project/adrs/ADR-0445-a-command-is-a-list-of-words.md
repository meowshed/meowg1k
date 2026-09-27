---
id: ADR-0445
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2008]
supersedes: []
---

# 0445. A command is a list of words, never one string

## Decision

A command is a list of words, never one string, and the error says why rather
than just refusing (from https://github.com/retran/meowg1k/pull/135, high).

What works now: `shell.run` and `shell.capture` take a list of strings, run the
first word as the program with the rest as its arguments and no shell between,
and refuse any other value with "takes a list of words ... a policy cannot
judge a command line it never saw the shape of" (from
crates/meow-star/src/capability.rs:316 and
crates/meow-star/src/capability.rs:336, high). `@std//git` builds its own word
list and goes through the same spawn (from
crates/meow-star/src/capability.rs:296, high). What still doesn't: when a tool
is offered to a model, the policy reads the `command` argument only when it is
a string, so a command given as a list of words reaches the `commands` selector
as no command at all (from crates/meow-star/src/run.rs:557 and
crates/meow-policy/src/policy.rs:321, medium).

## Why

A string would be split by a shell, and `R-POLICY-006` asks the policy to judge
a command line, which it can't do for a line it never saw the shape of (from
https://github.com/retran/meowg1k/pull/135, high). The pull request's
`R-POLICY-006` is the multi-path rule in the spec at that commit; the
obligation it describes, judging the full command line, is `R-POLICY-004`, which
is what `addresses:` names (from `git show 7bd211c:docs/spec/policy.md`, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Do nothing: accept a command as one string and run it with `sh -c`, as v0.2.x's `shell` module did | Pipes, `&&` and redirection work as a user types them, in one argument (from v0.2.1:internal/core/starlark/module_shell.go:32, high) | A shell splits it, and the policy can't judge a line it never saw the shape of (from https://github.com/retran/meowg1k/pull/135, high) |
| Accept both, splitting a string into words without a shell | A one-word command such as `"ls"` reads naturally, with no list around it (reasoned from crates/meow-star/src/capability.rs:326, low) | The source doesn't weigh it; the error exists so a string is refused with its reason, which a silent split would hide (from https://github.com/retran/meowg1k/pull/135, high) |

A third option isn't named by the code, the history or the design documents,
so the table stops at two.

## What it costs

A handler that needs a pipe, `&&` or a redirect has to call a shell itself,
`run(["sh", "-c", "..."])`, and so writes the one-string form it was meant to
avoid (reasoned from crates/meow-star/src/capability.rs:326, low).

## What would reverse it

The `commands` selector changing to match word lists in place of one string,
or `shell.run` being judged by policy as a string it passes to a shell, either
of which removes the shape the list preserves (reasoned from
docs/spec/policy.md `R-POLICY-004`, low).

## Consequences

- `run("echo hello && rm -rf /")` fails with the reason instead of running
  (from crates/meow-star/tests/running.rs:951, high).
- A handler's `shell.run` isn't judged by policy at all: policy governs what a
  model decided, not what a script did (from docs/spec/policy.md, Decisions,
  "Policy governs what a model decided, not what a script did", high).

## How I will know it was realised

`crates/meow-star/tests/running.rs` `shell_refuses_a_command_as_one_string`
passes: a handler passing one string gets an error containing "takes a list of
words" and "never saw the shape of" (from
crates/meow-star/tests/running.rs:951, high).
`shell_runs_a_command_and_reports_it` in the same file pins the list form (from
crates/meow-star/tests/running.rs:919, high).

## What this does not settle

- How a model-facing tool whose `command` argument is a list is judged by a
  `commands` rule; today the list isn't joined, so no `commands` selector
  matches it (from crates/meow-star/src/run.rs:557 and
  crates/meow-policy/src/policy.rs:321, medium).
- The environment a command inherits: `shell.run` sets the working directory
  and nothing else (from https://github.com/retran/meowg1k/pull/135, "What is
  not covered", high).
