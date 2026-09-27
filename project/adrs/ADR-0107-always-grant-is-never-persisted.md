---
id: ADR-0107
artifact: adr
status: approved
revised: 2026-09-27
addresses: [REQ-2026, REQ-2027, REQ-2842]
supersedes: []
---

# 0107. An "always" approval is never written back to `meow.star`

## Decision

The approval prompt's `always` answer lasts for the current process and is
never written back to `meow.star` (from crates/meow-policy/src/prompt.rs,
REQ-2026 and REQ-2842, high). The design document said "session-scoped"; the
code and both requirements scope it to the process, so a permission granted in
a hurry doesn't outlive the terminal it was granted in (from
crates/meow-policy/src/prompt.rs, high).

## Why

Persisting a grant from inside a prompt is how permission systems rot (from
docs/design/0.3.0-tui.md section 7, high): the next run inherits a decision
nobody remembers making (from crates/meow-policy/src/policy.rs:153-158, high).
A permission granted in a hurry shouldn't outlive the terminal it was granted
in (from crates/meow-cli/src/ask.rs:62-67, high).

## Alternatives

| Option | Better at | Why it lost |
| --- | --- | --- |
| Write an "always" grant back into `meow.star` | The user answers a question once and never again, in any later run (reasoned from crates/meow-policy/src/prompt.rs:80-86, low) | Persisting a grant from inside a prompt is how permission systems rot (from docs/design/0.3.0-tui.md section 7, high) |
| Do nothing: offer only `once`, `deny` and `stop` | No grant outlives a single call, so nothing can rot (reasoned from crates/meow-policy/src/prompt.rs:80-86, low) | The prompt has to offer once, always, deny and stop (from docs/spec/tui.md [R-TUI-061], high); without `always` a user answers the same question every step (reasoned from crates/meow-policy/src/prompt.rs:43-46, low) |
| Keep an "always" grant with the session, so a resumed session inherits it | A resumed run doesn't ask again for what was granted before (reasoned from docs/design/0.3.0-tui.md section 7, low) | A grant written into the session log is still a grant written down, and [R-POLICY-023] lets a grant apply for the current process only (reasoned from docs/spec/policy.md [R-POLICY-023], low) |

## What it costs

A user answers the same prompt again in every new process, including one that
resumes the session where the grant was made, because grants start empty (from
crates/meow-policy/tests/spec.rs:324-350, high). A user who wants the call
allowed for good has to edit the policy, which the prompt helps with by naming
the rule (from docs/design/0.3.0-tui.md section 7, high).

## What would reverse it

- A way to write a grant that a person reviews as a change to `meow.star`, such
  as a proposed edit shown as a diff, so the grant isn't made from inside a
  prompt (reasoned from docs/design/0.3.0-tui.md section 7, low). While the only
  way to persist is from the prompt, the argument holds.

## Consequences

The prompt names the matching rule, so the user learns why they're asked and can
narrow the policy afterwards by editing it (from docs/design/0.3.0-tui.md
section 7, high). Grants live in one place, the terminal approver's set, and
nothing writes that set to a file (from crates/meow-cli/src/ask.rs:62-67, high).
A grant is an input to evaluation, so the same call, policy and grants give the
same decision (from docs/spec/policy.md [R-POLICY-013], high), and a grant can't
turn a `deny` into anything else (from crates/meow-policy/tests/spec.rs:351-353,
high).

## How I will know it was realised

1. `a_session_grant_lives_and_dies_with_the_process` in
   crates/meow-policy/tests/spec.rs passes: a grant allows the call and says it
   came from a session grant, and a fresh `Grants` asks again (from
   crates/meow-policy/tests/spec.rs:324-350, high).
2. `a_grant_cannot_undo_a_deny` in crates/meow-policy/tests/spec.rs passes (from
   crates/meow-policy/tests/spec.rs:351-353, high).
3. After answering `always`, `git status` shows `meow.star` and every other
   workspace file unchanged (reasoned from docs/spec/policy.md [R-POLICY-023],
   medium).

## What this does not settle

- Whether a process that resumes a session should inherit its grants. It
  doesn't today: each new `Runtime` starts with empty grants, so a resumed
  session asks again (from crates/meow-cli/src/ask.rs:62-67 and
  crates/meow-star/src/run.rs:366, high).
