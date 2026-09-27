---
id: BUG-0124
artifact: bug
status: approved
severity: major
found: 2026-09-27
revised: 2026-09-27
issue:
---

# Editing a handler's body keeps trust, although the body holds the authority

The trust fingerprint covers agents, packages, tools, commands and policy
rules. A handler runs `@std//shell`, `@std//fs` and `@std//http` with no policy
in the way, so a pulled change that rewrites a handler to run any program keeps
the trust given to the old one and runs without a question.

## Reproduction

Seen at `40b4469` on branch `docs/onboard-record`, whose uncommitted changes
touch only documentation and comments; `cargo build -p meow-cli`, `HOME` set to
a scratch directory.

1. Create `.meow/meow.star`:

   ```python
   def go(ctx):
       ctx.out.write("ran")
       return "true"
   meow.command(meow.tool(name = "go", about = "s", run = go))
   ```

2. Run `meow trust`, then `meow go`. It prints `ran`.
3. Replace the file with one that adds `load("@std//shell", "capture")` and
   makes `go` call `capture(["touch", "pwned"])`.
4. Run `meow go`. No trust question is asked, and `pwned` exists afterwards.

## What the system does

`Declared::of` at `crates/meow-cli/src/trust.rs:33-72` records the declared
names and rules, and `fingerprint` at `trust.rs:80-92` hashes those lines. Its
comment says "a handler's body cannot widen it without widening one of these",
and step 4 shows that it can. The capability modules aren't behind policy,
`crates/meow-star/src/capability.rs:9-13` and
`crates/meow-star/src/capability_http.rs:19-22`. The test
`editing_a_handler_does_not_ask_again`, `crates/meow-cli/tests/trust.rs:180-203`,
pins the behaviour.

## What it should do, and why

No requirement covers it. REQ-1229 asks again when declarations change, and the
vision's first goal says a `.meow/` directory "is code with tool access, so the
first run shows what it declares and asks once, and asks again when that
changes". A new `@std//shell` load, or a changed handler, widens what the
workspace does as much as a new tool does, so trust should cover it.

## Triage

The defect enters in `meow-cli`'s trust fingerprint, by ADR-0458's choice. No
requirement is violated, so it routes to the requirements step: decide whether
trust covers the capability modules a workspace loads, or the text of `.meow/`.
It's major, and not critical, because no stated promise covers handler bodies;
but a `git pull` is enough to run new code with the person's authority, which
is what the trust prompt exists to stop.

## Closed by

A test in `crates/meow-cli/tests/trust.rs`, named for example
`loading_a_new_capability_module_asks_again`, running the reproduction above and
expecting the question, with `editing_a_handler_does_not_ask_again` changed to
match whatever the requirement decides.
