---
id: onboarding
artifact: onboarding
status: approved
revised: 2026-09-27
---

<!-- Written to the writing standard meow-prose ships: lead with what was found,
give each figure its source, and state a gap as plainly as a finding. -->

# Onboarding meowg1k

The record under `project/` now holds the vision, 11 specifications, 626 draft
requirements, 182 draft decisions and 87 defects found while checking them
against the code. Every recovered statement names its source and a confidence.
The legacy `docs/spec/`, `docs/design/` and `docs/philosophy.md` are removed:
their requirements, decisions and principles live in the record, and git keeps
the files. The owner asked for every gap to be researched and answered during
onboarding, so the Gaps section records answers, not questions. Every
requirement and decision stays a draft until the owner approves it.

## Verbs

All five verbs resolve, from `meow-verbs status` (0.3.0):

| Verb | Command |
| --- | --- |
| format | `mise run fmt-check` |
| lint | `mise run check` |
| check | `mise run typecheck` |
| test | `mise run test` |
| build | `mise run build` |

`mise run all` is the gate, and CI's `gates-agree` job fails when it and CI list
different checks. The `check` verb's task, `typecheck`, sits outside `all`,
because clippy under the `lint` verb already type-checks every target.

## Conventions

Each convention is what the repository did, counted on `main` on 2026-09-27, and
each is adopted in `.meowpaw/profile.toml` or `CLAUDE.md`, because every one
held without exception or had one clear majority.

- Commit subjects: 149 of 149 are conventional commits of 72 characters or
  fewer, over nine types (feat 58, build 21, fix 15, docs 15, ci 15, chore 14,
  refactor 5, spec 4, test 2). Adopted in `[commits]`.
- Sign-off: 149 of 149 carry `Signed-off-by`. Adopted as a required trailer.
- Signatures: 149 of 149 are signed; 146 verify locally and 3 are GitHub's
  squash merges. Adopted as `require_signatures = true`.
- Merging: GitHub allows squash merges only. Kept.
- Branch names: 61 of 105 merged pull requests by a person use `<type>/<slug>`;
  the 33 `retran/` branches are all from the Go era. Adopted in `CLAUDE.md`
  `reviewable_history`.
- `Closes #n`: 2 of 149 messages. Kept optional, because the squash title
  carries the pull request number.
- Licence header: 123 of 123 Rust files. Kept in `CLAUDE.md` `code_conventions`.
- Requirement tracing: all 315 `R-*` identifiers were cited under `crates/`. All
  1,056 citations were rewritten to the `REQ-*` identifiers each one split into,
  using the source line every requirement file carries; the withdrawn
  `R-SESSION-030` now points at ADR-2202, which records the rejected design.

## Documents

| Document | Outcome | Where, or why |
| --- | --- | --- |
| `docs/spec/*.md` (10 files and a README) | migrated | Removed; SPC-1000 to SPC-2800, REQ-1000 to REQ-2859, ADR-1000 to ADR-2806 |
| `docs/design/*.md` (6 files) | migrated | Removed; ADR-0100 to ADR-0121 and ADR-0600 to ADR-0623; the plan's milestones are executed history git keeps |
| `docs/philosophy.md` | migrated | Removed; folded into the vision's ranked quality goals and their enforcement table, at the owner's instruction |
| `docs/vision.md` | migrated | Rewritten as `project/vision.md` |
| `docs/README.md` | migrated | Rewritten as `project/README.md`, the record's index |
| `CLAUDE.md` | cited | The constitution; its record paths, `R-*` examples and the `miette` claim were corrected to match the record and the code |
| `README.md` | cited | Front page; its documentation links now point at `project/` |
| `CONTRIBUTING.md` | cited | Contributor workflow; its requirement section now describes `REQ-*` files |
| `CHANGELOG.md` | cited | Release history, kept as written, including its mention of the removed design document |
| `CODE_OF_CONDUCT.md` | cited | Community policy, outside the record |
| `SECURITY.md` | cited | Disclosure policy, outside the record |
| `LICENSE_HEADER.txt` | cited | The header template the `code_conventions` principle and the pre-commit hook use |
| `.github/ISSUE_TEMPLATE/bug_report.md` | cited | Forge template, outside the record |
| `.github/ISSUE_TEMPLATE/feature_request.md` | cited | Forge template, outside the record |
| `.meow/agents/assistant.md` | cited | The product's own workspace, which the record describes and doesn't hold |
| `.meow/agents/committer.md` | cited | The product's own workspace, which the record describes and doesn't hold |
| `.meow/agents/reviewer.md` | cited | The product's own workspace, which the record describes and doesn't hold |
| `.meow/lib/repository.md` | cited | The product's own workspace; it now points its agents at `project/` |
| `.meow/lib/style.md` | cited | The product's own workspace, which the record describes and doesn't hold |
| Forge history, #102 to #172 | migrated | REQ-1073 to REQ-1077 and REQ-1230 to REQ-1233 from open issues #166 to #168; ADR-0400 to ADR-0469 from pull requests #109 to #163 |
| Forge history, #1 to #101 | discarded | Choices and obligations for the Go implementation, which no longer exists and no current requirement covers |

## Gaps

Each question the first draft of this report asked, with the answer the research
found. Where an answer is a choice made during onboarding, not a fact, it says
so.

Contradictions:

1. "Always" lasts for the process, not the session. The code
   (`crates/meow-policy/src/prompt.rs`) and REQ-2026 and REQ-2842 agree; only
   the removed design document said session. ADR-0107 is amended.
2. An omitted budget axis isn't a contradiction. The engine's `Budget` leaves an
   unset axis unbounded (REQ-1012), and a Starlark declaration fills every axis
   from the default before the engine sees it (REQ-2528, ADR-0418). REQ-1012 now
   says so.
3. The session identifier's tail is entropy from the clock and the process id,
   deliberately without a random generator (`crates/meow-cli/src/wire.rs:922`).
   REQ-2220 said "random" and is amended; ADR-0438 records what that costs.
4. `toml.encode` refuses a null, because TOML has none. REQ-2432 is narrowed to
   values a format can represent, and REQ-2529 states the refusal.
5. Declaration order: REQ-2479 governs references by name, which resolve late;
   `meow.command` takes a Starlark value, which ordinary scoping binds first.
   REQ-2479 and ADR-0415 now say so.
6. Credentials: REQ-1204 states what the binary does today, and REQ-1230 to
   REQ-1233 state issue #166's change. Both stay drafts; the new ones supersede
   REQ-1204 to REQ-1208 when #166 lands.
7. Cost (#168): the price goes in the model declaration (ADR-0601, REQ-2530,
   REQ-2262, REQ-1078), chosen because it keeps the cost axis useful and the
   issue itself calls it the more useful fix. A choice made during onboarding.
8. Type names follow the code: `Outcome`, `Sink` and `Engine::run` replace the
   removed spec's `AgentOutcome`, `EventSink` and `AgentRun` in REQ-1000,
   REQ-1002, REQ-1060, SPC-1000 and ADR-1004.
9. Seven provider kinds ship, listed in `KINDS` in
   `crates/meow-cli/src/wire.rs`; the vision says seven.

What the sources didn't record:

1. Every decision now has its cost, reversal condition, consequences, check and
   open questions, each cited to code, a test, a pull request or a design
   section. Answers that are reasoning over that evidence are marked `low`,
   mostly "better at" cells and reversal conditions; those are the ones to read
   first when approving.
2. Each alternatives table carries a "Do nothing" row where keeping the prior
   state was a real option, and up to three options where the history names
   them. Where it names fewer, the table says so.
3. The quality goals are ranked in the vision: security first, configuration as
   code second. A choice made during onboarding, reasoned from `CLAUDE.md`.
4. The scripting audience's alternative today is shell scripts that pipe text to
   a model's API, which issue #1 set out to replace.

Choices no requirement covered now have one: REQ-3000 to REQ-3011 and SPC-3000
for the crate boundaries, and 43 further requirements with ADR-0600 to ADR-0623
for the Starlark API design, sessions, HTTP transport, git, fs, packages and the
uncovered module calls. Renovate's approval for major bumps and issue #7's
branch-aware indexing are repository practice and an idea, not behaviour, so
they have no requirement. SPC-2600's internal Rust API row is removed, because a
crate's API isn't a user-observable boundary.

Mappings: every `addresses:` list was checked against the code while the
decisions were completed; 29 were corrected, and each decision names what it
settles. REQ-2415's judge is the repository owner. Body cross-references keep
the `R-*` identifier where they quote their source, which the file names.

What the recovery couldn't check: `project/research/RES-0001-synthesis.md` stays
missing, because the repository keeps no research; the first research step
writes it. REQ-1073 to REQ-1078, REQ-1230 to REQ-1233, REQ-2262, REQ-2530 and
REQ-2531 describe unbuilt behaviour, so no specification states them yet.

Defects: completing the decisions against the code turned up 88 suspected
defects. After each was checked in the code, and 20 of them by running the
binary or a scratch crate, 87 records stand in `project/bugs/`: 3 critical, 33
major and 51 minor. The critical ones are BUG-0017 (with no declared policy,
every tool call runs unchecked), BUG-0115 (a sensitive argument reaches the
session log in the clear) and BUG-0116 (`meow session export` and `show` never
redact). Eleven suspicions didn't hold and have no record. None is fixed here,
because onboarding changes no behaviour.

## Adoption

1. Approve the drafts: the requirements by topic, then the decisions that
   address them. The repository keeps working throughout, because nothing it
   runs reads the record. Once done, `paw status` shows requirements in
   force and coverage becomes meaningful.
2. Triage `project/bugs/`, starting with the critical ones, and fix each as its
   own task. Once done, each defect's regression test passes and it closes.
3. Run `paw onboarding remove` to finish. It removes each document this report
   marks migrated, superseded or discarded and keeps each one marked cited.
   Once done, the record describes only the present.
