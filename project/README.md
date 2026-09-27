---
id: index
artifact: index
status: live
revised: 2026-09-27
---

# The record

This directory is meowg1k's record: what the project is for, what the binary
must do, why it's built the way it is, and what it does now.

- [vision.md](vision.md) is what meowg1k is, who it is for, what it deliberately
  isn't, and the eleven goals it's held to, ranked. Read it first if you haven't
  used the tool.
- [requirements/](requirements/README.md) holds one obligation per file,
  `REQ-NNNN`, in topic blocks of 200. Every behaviour the binary has traces to
  one, and a change that no requirement describes needs a requirement first.
  Tests name the requirement they check in a doc comment.
- [adrs/](adrs/README.md) holds one decision per file, `ADR-NNNN`, with the
  requirements it settles, the alternatives it beat and why, and what would
  reverse it.
- [specs/](specs/) states what each part of the binary does now, citing the
  requirement behind every statement.
- [bugs/](bugs/) holds one defect per file, `BUG-NNNN`, each naming the
  requirement it breaks.

`.meow/` in this repository is meowg1k configured to work on itself: three
markdown agents, two shared prompt fragments, and the commands in
`.meow/meow.star`. It is the worked example that has to keep working, because it
is what the maintainers run.

## Specifications

A specification is listed after every specification it cites.

- [SPC-1000](specs/SPC-1000-agent-loop.md): The agent loop in meow-agent
- [SPC-1200](specs/SPC-1200-authentication-and-trust.md): Authentication and
  trust
- [SPC-1400](specs/SPC-1400-index.md): The index that makes a workspace
  searchable by meaning
- [SPC-1600](specs/SPC-1600-llm-provider-contract.md): The model provider
  contract
- [SPC-2000](specs/SPC-2000-policy.md): The policy layer that decides whether a
  tool call runs
- [SPC-2200](specs/SPC-2200-session.md): The session log of an agent run
- [SPC-2400](specs/SPC-2400-starlark-surface.md): The Starlark surface users
  write against
- [SPC-1800](specs/SPC-1800-packages.md): Packages: Starlark a workspace did not
  write
- [SPC-2600](specs/SPC-2600-workspace-store.md): The workspace store
- [SPC-2800](specs/SPC-2800-terminal.md): The terminal renderers and the command
  line
- [SPC-3000](specs/SPC-3000-crate-structure.md): The workspace's crate structure

## Defects

Each defect has a GitHub issue, named after the severity.

- [BUG-0001](bugs/BUG-0001-embedding-with-a-new-model-relabels-old-vectors.md)
  (major, #174): Embedding with a new model relabels the old vectors as the new
  model's
- [BUG-0002](bugs/BUG-0002-only-http-413-is-read-as-a-chunk-too-large.md)
  (minor, #175): Only HTTP 413 is read as a chunk too large, so other refusals
  don't name the file
- [BUG-0003](bugs/BUG-0003-chunk-too-large-reports-the-chunker-limit.md) (minor,
  #176): A chunk too large reports the chunker's limit as the model's
- [BUG-0004](bugs/BUG-0004-index-clear-deletes-a-key-value-entry.md) (minor,
  #177): `meow index clear` deletes the `index.model` entry from the key-value
  store
- [BUG-0005](bugs/BUG-0005-no-provider-call-is-retried.md) (major, #178): No
  provider call is retried, so a 429 or 503 fails the run at once
- [BUG-0006](bugs/BUG-0006-anthropic-overloaded-529-is-classified-fatal.md)
  (minor, #179): Anthropic's 529 "overloaded" is classified `Fatal`
- [BUG-0007](bugs/BUG-0007-thinking-is-not-stored-in-the-session-log.md) (minor,
  #180): Thinking is not stored in the session log
- [BUG-0008](bugs/BUG-0008-anthropic-resends-thinking-without-its-signature.md)
  (minor, #181): Anthropic sends thinking back without its signature
- [BUG-0009](bugs/BUG-0009-anthropic-translates-a-cache-hint-only-on-system.md)
  (minor, #182): Anthropic translates a cache hint only on a system message
- [BUG-0010](bugs/BUG-0010-openai-provider-assigns-no-id-to-a-call-without-one.md)
  (major, #183): The OpenAI-shaped provider assigns no identifier to a tool call
  without one
- [BUG-0011](bugs/BUG-0011-concurrent-requests-can-each-renew-the-credential.md)
  (minor, #184): Concurrent requests can each renew an expired exchanged
  credential
- [BUG-0012](bugs/BUG-0012-a-renewal-without-expiry-renews-every-request.md)
  (minor, #185): A renewal answer without `expires_at` makes every request renew
- [BUG-0013](bugs/BUG-0013-a-package-file-can-load-the-host-workspace.md)
  (major, #186): A package file can load the host workspace's files with `//`
- [BUG-0014](bugs/BUG-0014-a-package-sibling-loads-by-package-name-not-relative-path.md)
  (minor, #187): A package loads its own sibling by package name, not by a
  relative path
- [BUG-0015](bugs/BUG-0015-the-engine-ignores-the-paths-policy-denied.md)
  (major, #188): The engine ignores the paths a policy verdict denied
- [BUG-0016](bugs/BUG-0016-one-denied-path-denies-a-whole-read.md) (minor,
  #189): An explicit deny on one path denies a whole read
- [BUG-0017](bugs/BUG-0017-with-no-declared-policy-every-tool-call-runs.md)
  (critical, #190): With no declared policy, every tool call runs unchecked
- [BUG-0018](bugs/BUG-0018-an-agent-policy-replaces-the-workspace-policy.md)
  (major, #191): An agent's policy replaces the workspace's, so it can allow
  what the workspace denies
- [BUG-0019](bugs/BUG-0019-an-approval-prompt-timeout-cannot-be-configured.md)
  (minor, #192): An approval prompt's timeout can't be configured
- [BUG-0020](bugs/BUG-0020-the-session-heartbeat-is-written-only-at-start.md)
  (major, #193): The session heartbeat is written only when the session starts
- [BUG-0021](bugs/BUG-0021-a-dead-session-is-never-finished.md) (major, #194): A
  session whose process died is never finished
- [BUG-0022](bugs/BUG-0022-a-compaction-records-message-positions-not-sequence-numbers.md)
  (major, #195): A compaction records message positions, not the sequence
  numbers it supersedes
- [BUG-0023](bugs/BUG-0023-a-failed-session-write-is-silently-dropped.md)
  (major, #196): A failed session write is dropped without a word
- [BUG-0024](bugs/BUG-0024-event-payloads-are-not-addressed-by-blake3.md)
  (major, #197): Event payloads are stored inline, not addressed by their BLAKE3
  hash
- [BUG-0025](bugs/BUG-0025-doctor-does-not-report-the-unencrypted-database.md)
  (minor, #198): `meow doctor` doesn't report that the database is unencrypted,
  or its path
- [BUG-0026](bugs/BUG-0026-index-graph-files-are-not-owner-only.md) (major,
  #199): The index's graph files are created readable by everyone
- [BUG-0027](bugs/BUG-0027-a-workspace-command-exits-only-0-1-or-9.md) (major,
  #200): A workspace command exits only 0, 1 or 9, whatever the agent's stop
  reason
- [BUG-0028](bugs/BUG-0028-xml-parse-drops-entities-and-cdata.md) (major, #201):
  `xml.parse` drops entity references and CDATA from an element's text
- [BUG-0029](bugs/BUG-0029-store-put-returns-a-tuple-as-a-list.md) (minor,
  #202): `store.put` of a tuple comes back from `store.get` as a list
- [BUG-0030](bugs/BUG-0030-search-text-ignores-the-declared-walk-when-the-index-cannot-open.md)
  (minor, #203): `search.text` walks with defaults when the declared index can't
  be built
- [BUG-0031](bugs/BUG-0031-meow-policy-explain-does-not-exist.md) (major, #204):
  `meow policy explain` doesn't exist
- [BUG-0032](bugs/BUG-0032-path-base-returns-a-backslash-on-unix.md) (minor,
  #205): `path.base` returns a backslash on Unix
- [BUG-0033](bugs/BUG-0033-http-reads-the-whole-body-before-applying-the-cap.md)
  (major, #206): `http` reads the whole response body before cutting it to
  `max_bytes`
- [BUG-0034](bugs/BUG-0034-csv-parse-drops-a-column-whose-header-repeats.md)
  (minor, #207): `csv.parse` silently drops a column whose header name repeats
- [BUG-0035](bugs/BUG-0035-prompt-open-docs-say-the-prompt-sits-in-the-live-region.md)
  (minor, #208): `Renderer::prompt_open`'s documentation says the prompt
  occupies the live region
- [BUG-0036](bugs/BUG-0036-meow-index-doc-comment-sits-on-meow-package.md)
  (minor, #209): `meow.index`'s doc comment sits on `meow.package`
- [BUG-0037](bugs/BUG-0037-decisions-name-a-meow-session-resume-the-binary-lacks.md)
  (minor, #210): Two decisions name a `meow session resume` subcommand the
  binary doesn't have
- [BUG-0038](bugs/BUG-0038-a-tool-acts-on-the-models-path-not-the-judged-one.md)
  (major, #211): A tool acts on the model's raw path, not the path the policy
  judged
- [BUG-0039](bugs/BUG-0039-tokens-and-cost-are-checked-before-and-charged-after.md)
  (minor, #212): Tokens and cost are checked before a call and charged after, so
  concurrent calls can overspend
- [BUG-0040](bugs/BUG-0040-an-event-and-its-side-row-commit-separately.md)
  (minor, #213): A `Usage` or `Compaction` event and its side-table row commit
  separately
- [BUG-0041](bugs/BUG-0041-answering-stop-ends-a-report-run-denied-without-a-requirement.md)
  (minor, #214): Answering "stop" ends a run `denied` under a `report` policy,
  and no requirement says so
- [BUG-0042](bugs/BUG-0042-a-streamed-error-loses-the-quota-signal-and-retry-after.md)
  (minor, #215): A streamed request's HTTP error loses the provider's quota
  signal and `Retry-After`
- [BUG-0043](bugs/BUG-0043-an-http-date-retry-after-is-ignored.md) (minor,
  #216): A `Retry-After` in HTTP-date form is ignored, and Gemini then reads a
  rate limit as a spent quota
- [BUG-0044](bugs/BUG-0044-a-server-retry-after-is-not-capped.md) (minor, #217):
  A server's `Retry-After` is used without a ceiling
- [BUG-0045](bugs/BUG-0045-compaction-counts-bytes-and-the-chunker-counts-characters.md)
  (minor, #218): Compaction estimates tokens from bytes, and the chunker counts
  characters
- [BUG-0046](bugs/BUG-0046-an-undeclared-reference-fails-with-no-location.md)
  (minor, #219): A reference to an undeclared name fails with no file, line or
  column
- [BUG-0047](bugs/BUG-0047-an-argument-glob-pattern-reaches-the-model-as-a-regex.md)
  (minor, #220): An argument's glob `pattern` reaches the model as a JSON Schema
  regular expression
- [BUG-0048](bugs/BUG-0048-git-checks-arguments-before-refusing-during-declaration.md)
  (minor, #221): `@std//git` checks its arguments before refusing a call during
  declaration
- [BUG-0049](bugs/BUG-0049-the-plain-renderer-prints-no-model-text-in-a-live-run.md)
  (major, #222): The plain renderer prints none of the model's text in a live
  run
- [BUG-0100](bugs/BUG-0100-a-sub-agent-at-the-depth-limit-is-silently-left-out.md)
  (major, #223): A sub-agent at the depth limit is silently left out of its
  caller's tools
- [BUG-0101](bugs/BUG-0101-agent-run-from-a-handler-adds-no-nesting-depth.md)
  (minor, #224): `agent.run` from a tool handler adds no nesting depth
- [BUG-0102](bugs/BUG-0102-a-missing-credential-on-the-agent-path-reads-as-undeclared.md)
  (major, #225): A missing credential on the agent path reads as an undeclared
  provider and exits 9
- [BUG-0103](bugs/BUG-0103-a-command-named-pkg-or-help-panics-the-binary.md)
  (major, #226): A workspace command named `pkg` or `help` panics the binary
- [BUG-0104](bugs/BUG-0104-retention-by-database-size-cannot-be-configured.md)
  (major, #227): Retention by total database size can't be configured
- [BUG-0105](bugs/BUG-0105-a-size-limit-deletes-every-unprotected-session.md)
  (major, #228): A retention size limit deletes every unprotected session
- [BUG-0106](bugs/BUG-0106-a-dry-run-states-its-divergence-once-per-tool.md)
  (minor, #229): A dry run states its divergence once per planned tool, not once
  per run
- [BUG-0107](bugs/BUG-0107-meow-parallel-runs-its-invocations-one-after-another.md)
  (major, #230): `meow.parallel` runs its invocations one after another
- [BUG-0108](bugs/BUG-0108-the-policy-event-omits-the-rule-and-the-source.md)
  (major, #231): The `Policy` event omits the rule and records the decision
  before the answer
- [BUG-0109](bugs/BUG-0109-a-broken-sink-is-never-reported.md) (minor, #232): A
  broken sink's error is kept and never reported
- [BUG-0110](bugs/BUG-0110-a-tool-handler-runs-on-a-fresh-ledger.md) (major,
  #233): A Starlark tool's handler runs on a fresh ledger, outside its caller's
  budget
- [BUG-0111](bugs/BUG-0111-the-schema-attempt-count-cannot-be-configured.md)
  (minor, #234): The schema retry count is a constant, not a configured value
- [BUG-0112](bugs/BUG-0112-the-engine-outcome-carries-no-session-identifier.md)
  (minor, #235): The engine's `Outcome` carries no session identifier
- [BUG-0113](bugs/BUG-0113-a-negation-in-a-nested-meowignore-is-ignored.md)
  (minor, #236): A negation in a nested `.meowignore` can't re-include a
  git-ignored file
- [BUG-0114](bugs/BUG-0114-a-session-value-is-not-restored-on-resume.md) (minor,
  #237): A value set with `ctx.session.set` is gone when the session is resumed
- [BUG-0115](bugs/BUG-0115-a-sensitive-argument-is-written-to-the-log-in-the-clear.md)
  (critical, #238): A sensitive argument is written to the session log in the
  clear
- [BUG-0116](bugs/BUG-0116-session-export-never-redacts-a-sensitive-value.md)
  (critical, #239): `meow session export` and `show` never redact a sensitive
  value
- [BUG-0117](bugs/BUG-0117-the-session-identifier-repeats-bits-it-calls-entropy.md)
  (minor, #240): The session identifier repeats 16 bits it counts as entropy
- [BUG-0118](bugs/BUG-0118-a-resumed-run-loses-the-tool-calls-and-results.md)
  (major, #241): A resumed run loses the earlier run's tool calls and results
- [BUG-0119](bugs/BUG-0119-a-command-given-as-a-list-escapes-the-commands-rules.md)
  (major, #242): A `command` argument given as a list escapes every `commands`
  rule
- [BUG-0120](bugs/BUG-0120-gemini-strips-a-property-named-like-a-refused-keyword.md)
  (minor, #243): Gemini's schema filter strips a property named `default`
- [BUG-0121](bugs/BUG-0121-the-voyage-provider-does-not-implement-generation.md)
  (minor, #244): The Voyage provider doesn't implement generation
- [BUG-0122](bugs/BUG-0122-no-test-reaches-the-store-that-will-not-open.md)
  (minor, #245): No test reaches the `store` that won't open, and the binary
  fails before it can
- [BUG-0123](bugs/BUG-0123-no-test-checks-that-text-search-obeys-the-ignore-files.md)
  (minor, #246): No test checks that `search.text` and `search.files` obey the
  ignore files
- [BUG-0124](bugs/BUG-0124-editing-a-handler-body-keeps-trust.md) (major, #247):
  Editing a handler's body keeps trust, although the body holds the authority
- [BUG-0125](bugs/BUG-0125-meow-pkg-contacts-a-workspace-named-host-without-trust.md)
  (major, #248): `meow pkg update` contacts a host an untrusted workspace names
- [BUG-0126](bugs/BUG-0126-agreeing-to-trust-exits-0-without-running-the-command.md)
  (major, #249): Agreeing to trust exits 0 without running the command or saying
  so
- [BUG-0127](bugs/BUG-0127-a-copilot-provider-under-another-name-cannot-log-in.md)
  (minor, #250): A `copilot` provider declared under another name can't log in
  by device code
- [BUG-0128](bugs/BUG-0128-a-throttled-credential-renewal-is-fatal.md) (minor,
  #251): A throttled or unavailable credential renewal is fatal and blames the
  grant
- [BUG-0129](bugs/BUG-0129-concurrent-store-writes-share-one-temporary-file.md)
  (minor, #252): Two processes writing the credential store share one temporary
  file
- [BUG-0130](bugs/BUG-0130-the-trust-fingerprint-omits-the-locked-package-hash.md)
  (major, #253): The trust fingerprint omits the locked package hash
- [BUG-0131](bugs/BUG-0131-meow-pkg-fetch-does-not-replace-a-tampered-cache.md)
  (major, #254): `meow pkg fetch` doesn't replace a cached package whose
  contents changed
- [BUG-0132](bugs/BUG-0132-no-test-cancels-an-http-call.md) (minor, #255): No
  test calls `@std//http` with a cancelled run
- [BUG-0133](bugs/BUG-0133-meow-run-ignores-the-arguments-after-the-name.md)
  (minor, #256): `meow run <name>` ignores the arguments after the name
- [BUG-0134](bugs/BUG-0134-meow-session-depends-on-meow-policy.md) (minor,
  #257): `meow-session` depends on `meow-policy` for one constant
- [BUG-0135](bugs/BUG-0135-nothing-enforces-the-crate-dependency-graph.md)
  (minor, #258): Nothing enforces the crate dependency graph
- [BUG-0136](bugs/BUG-0136-a-package-fetch-reads-the-whole-body-before-the-cap.md)
  (minor, #259): A package fetch reads the whole body into memory before
  checking the cap
- [BUG-0137](bugs/BUG-0137-fs-glob-follows-links-and-loops-on-an-ancestor-link.md)
  (minor, #260): `fs.glob` follows linked directories and doesn't finish on two
  ancestor links
- [BUG-0200](bugs/BUG-0200-credentials-stored-in-plaintext-file.md) (major,
  #166): Credentials are stored in a plaintext file, not the platform secret
  store
- [BUG-0201](bugs/BUG-0201-request-rate-limit-missing.md) (major, #167): A
  workspace can't declare a request ceiling, so nothing limits request rate
- [BUG-0202](bugs/BUG-0202-cost-cap-never-fires.md) (major, #168): A declared
  cost cap never stops a run, because no model carries a price
