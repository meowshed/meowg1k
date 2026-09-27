# Requirements

One obligation per file. Each was recovered from `docs/spec/` or an open issue
on 2026-09-27 and is a draft until the owner approves it.

<!-- The generated topic lines list every identifier on one line. -->
<!-- markdownlint-disable MD013 -->
<!-- meow-flow index -->

626 requirements in all: 625 approved, 1 superseded.

| Identifier | What it requires | Status |
| --- | --- | --- |
| [REQ-1000](REQ-1000-every-run-returns-an-outcome.md) | Every run that starts MUST return an `Outcome`. No stop condition, cancellation included, may be reported as an error, because an error discards the transcript the run already paid for. | approved |
| [REQ-1001](REQ-1001-stop-reasons-are-exactly-six.md) | The stop reasons MUST be exactly: `finished`, `budget`, `cancelled`, `denied`, `tool_aborted`, and `failed`. | approved |
| [REQ-1002](REQ-1002-outcome-text-carries-last-output.md) | `Outcome.text` MUST carry the model's last text output, whatever the stop reason, including when that text is empty. | approved |
| [REQ-1003](REQ-1003-outcome-carries-transcript-usage-detail.md) | An outcome MUST carry the step transcript, the usage totals, the session identifier, and a detail string that explains the stop reason: which budget axis bound the run, which tool aborted it, which rule denied it. | approved |
| [REQ-1004](REQ-1004-no-tool-calls-stops-finished.md) | A model response with no tool calls MUST stop the run with `finished`, including when its text is empty. | approved |
| [REQ-1005](REQ-1005-empty-text-is-not-failure.md) | Empty text MUST NOT be treated as a failure. | approved |
| [REQ-1006](REQ-1006-detail-records-empty-model-text.md) | The outcome's detail MUST record that the model returned no text, so a caller is not handed a silent success. | approved |
| [REQ-1007](REQ-1007-policy-denial-stops-denied.md) | `denied` MUST be the stop reason when the run ends because policy denied a call and the error policy is `abort`. | approved |
| [REQ-1008](REQ-1008-tool-error-stops-tool-aborted.md) | `tool_aborted` MUST be the stop reason when the run ends because a tool returned an error and the error policy is `abort`. | approved |
| [REQ-1009](REQ-1009-budget-bounds-four-axes.md) | A budget MUST bound a run on four axes: total tokens, steps, wall-clock duration, and estimated cost. | approved |
| [REQ-1010](REQ-1010-run-stops-at-budget-axis.md) | A run MUST stop with `budget` as soon as any configured axis is reached. | approved |
| [REQ-1011](REQ-1011-outcome-names-binding-budget-axis.md) | The outcome of a run stopped with `budget` MUST name which axis bound it. | approved |
| [REQ-1012](REQ-1012-unset-budget-axis-is-unbounded.md) | An axis left unset on the engine's `Budget` MUST be unbounded. A Starlark agent declaration fills every axis before it reaches the engine, as REQ-2528 states (from crates/meow-star/src/agent.rs, high). | approved |
| [REQ-1013](REQ-1013-missing-budget-takes-default.md) | A spec with no budget at all MUST take the configured default rather than running unbounded. | approved |
| [REQ-1014](REQ-1014-sub-agent-spend-counts-against-caller.md) | A sub-agent's spend MUST count against its caller's remaining budget, transitively. | approved |
| [REQ-1015](REQ-1015-budget-reserved-before-call.md) | Budget MUST be reserved before a call, not checked before and charged after. | approved |
| [REQ-1016](REQ-1016-concurrent-invocations-cannot-overspend.md) | Concurrent invocations sharing one budget MUST NOT be able to overspend it by each observing the same remaining amount. | approved |
| [REQ-1017](REQ-1017-sub-agent-budget-within-remaining.md) | A sub-agent MUST NOT be given a budget larger than its caller's remaining budget. | approved |
| [REQ-1018](REQ-1018-budget-checked-each-call-result.md) | The budget MUST be checked before each model call and after each tool result, so a run cannot overshoot by a whole step. | approved |
| [REQ-1019](REQ-1019-default-budget-values.md) | The default budget MUST be 200,000 tokens, 40 steps, and 30 minutes, with cost unbounded. | approved |
| [REQ-1020](REQ-1020-validate-arguments-against-schema.md) | Before invoking a tool the engine MUST validate the arguments against the tool's schema. | approved |
| [REQ-1021](REQ-1021-omitted-required-argument-not-defaulted.md) | A required argument the model omitted MUST NOT be replaced with a default or a zero value. | approved |
| [REQ-1022](REQ-1022-engine-names-omitted-required-argument.md) | When the model omitted a required argument, the engine MUST return a message to the model naming the argument and its type. | approved |
| [REQ-1023](REQ-1023-run-continues-after-argument-correction.md) | When the model omitted a required argument, the engine MUST continue the run. | approved |
| [REQ-1024](REQ-1024-argument-correction-never-aborts-run.md) | An argument correction is not a tool error and MUST NOT abort the run under any error policy. | approved |
| [REQ-1025](REQ-1025-omitted-optional-argument-takes-default.md) | An optional argument the model omitted MUST take its declared default. | approved |
| [REQ-1026](REQ-1026-argument-without-default-stays-absent.md) | An optional argument the model omitted with no default MUST be absent rather than zeroed, so that absent is distinguishable from zero, empty, and false. | approved |
| [REQ-1027](REQ-1027-unknown-tool-returns-error.md) | A tool call naming a tool the agent was not given MUST return an error to the model. | approved |
| [REQ-1028](REQ-1028-unknown-tool-searches-no-registry.md) | A tool call naming a tool the agent was not given MUST NOT search any wider registry. | approved |
| [REQ-1029](REQ-1029-report-policy-returns-tool-error.md) | With error policy `report`, a tool error MUST be returned to the model as a tool result. | approved |
| [REQ-1030](REQ-1030-report-policy-continues-run.md) | With error policy `report`, the run MUST continue after a tool error. | approved |
| [REQ-1031](REQ-1031-abort-policy-stops-run.md) | With error policy `abort`, the run MUST stop on a tool error. | approved |
| [REQ-1032](REQ-1032-tool-calls-run-in-order.md) | Tool calls within one model response MUST execute in the order the model returned them. | approved |
| [REQ-1033](REQ-1033-run-accepts-cancellation-token.md) | Every run MUST accept a cancellation token. | approved |
| [REQ-1034](REQ-1034-run-checks-cancellation-token.md) | Every run MUST check its cancellation token before each model call, before each tool call, and while awaiting either. | approved |
| [REQ-1035](REQ-1035-cancelled-run-stops-cancelled.md) | A cancelled run MUST stop with `cancelled`. | approved |
| [REQ-1036](REQ-1036-cancelled-run-writes-finished-event.md) | A cancelled run MUST write its `Finished` event. | approved |
| [REQ-1037](REQ-1037-cancelled-run-returns-transcript.md) | A cancelled run MUST return the transcript accumulated so far. | approved |
| [REQ-1038](REQ-1038-parent-cancel-cancels-sub-agents.md) | Cancelling a parent MUST cancel its running sub-agents. | approved |
| [REQ-1039](REQ-1039-compact-above-context-fraction.md) | When the rebuilt message list exceeds the configured fraction of the model's context window, the engine MUST compact before the next model call. | approved |
| [REQ-1040](REQ-1040-compaction-keeps-recent-messages.md) | Compaction MUST keep the configured number of most recent messages verbatim. | approved |
| [REQ-1041](REQ-1041-compaction-records-session-event.md) | Compaction MUST record a `Compaction` session event per [R-SESSION-010]. | approved |
| [REQ-1042](REQ-1042-compaction-deletes-no-events.md) | Compaction MUST NOT delete events. | approved |
| [REQ-1043](REQ-1043-failed-compaction-fails-run.md) | A compaction that fails MUST fail the run rather than continuing with an over-long context that the provider will reject. | approved |
| [REQ-1044](REQ-1044-compaction-summarises-with-model-call.md) | Compaction MUST summarise the superseded range with a model call. | approved |
| [REQ-1045](REQ-1045-compaction-records-tokens-saved.md) | Compaction MUST record the number of tokens the summary saved. | approved |
| [REQ-1046](REQ-1046-compaction-never-drops-silently.md) | Compaction MUST NOT drop messages without summarising them, because a silent drop leaves the model confidently wrong about what it already knows. | approved |
| [REQ-1047](REQ-1047-compaction-accepts-own-model.md) | The compaction policy MUST accept a model of its own. | approved |
| [REQ-1048](REQ-1048-compaction-falls-back-agent-model.md) | The compaction policy MUST fall back to the agent's model when none is given. | approved |
| [REQ-1049](REQ-1049-sub-agent-runs-own-session.md) | An agent used as a tool MUST run in its own session, with the calling session recorded as its parent. | approved |
| [REQ-1050](REQ-1050-sub-agent-outcome-returned-as-result.md) | A sub-agent's outcome MUST be returned to the calling model as a tool result containing its text, its stop reason, and its parsed value when it declared an output schema. | approved |
| [REQ-1051](REQ-1051-refuse-sub-agent-beyond-depth.md) | The engine MUST refuse to start a sub-agent that would exceed the configured maximum nesting depth. | approved |
| [REQ-1052](REQ-1052-depth-refusal-is-tool-error.md) | The engine MUST report a refusal to start a sub-agent beyond the maximum nesting depth as a tool error rather than a panic. | approved |
| [REQ-1053](REQ-1053-sub-agent-excludes-caller-messages.md) | A sub-agent's message list MUST NOT contain its caller's messages. | approved |
| [REQ-1054](REQ-1054-only-task-and-arguments-cross.md) | The only information that crosses from a caller to a sub-agent MUST be the task and the arguments the caller passed, so that an agent behaves the same wherever it is called from. | approved |
| [REQ-1055](REQ-1055-run-invocations-concurrently-in-order.md) | The engine MUST accept a list of agent invocations and run them concurrently, returning results in the order given. | approved |
| [REQ-1056](REQ-1056-failing-invocation-spares-others.md) | One failing invocation MUST NOT abort the others. | approved |
| [REQ-1057](REQ-1057-failing-invocation-returns-outcome.md) | A failing invocation's result MUST be an outcome with a non-`finished` stop reason. | approved |
| [REQ-1058](REQ-1058-concurrent-invocations-share-budget.md) | Concurrent invocations MUST share the caller's budget. | approved |
| [REQ-1059](REQ-1059-exhausted-budget-stops-new-invocations.md) | The engine MUST stop starting new concurrent invocations once the budget is exhausted. | approved |
| [REQ-1060](REQ-1060-engine-emits-every-transition.md) | The engine MUST emit every observable transition to its `Sink`: run start and end, step start and end, text and thinking deltas, tool call start and end, policy decisions, and usage. | approved |
| [REQ-1061](REQ-1061-sink-can-decline-deltas.md) | A sink MUST be able to decline delta events. | approved |
| [REQ-1062](REQ-1062-declined-deltas-delivered-per-step.md) | When a sink declines delta events, the engine MUST deliver the completed text once per step instead, because a sink backed by a script callback would otherwise pay one call per token. | approved |
| [REQ-1063](REQ-1063-sink-errors-handled-identically.md) | An error returned by the sink MUST be handled identically for every event kind. | approved |
| [REQ-1064](REQ-1064-engine-stops-delivering-to-failed-sink.md) | When the sink returns an error, the engine MUST stop delivering to that sink. | approved |
| [REQ-1065](REQ-1065-engine-records-sink-error-note.md) | When the sink returns an error, the engine MUST record a `Note` event naming the error. | approved |
| [REQ-1066](REQ-1066-engine-continues-after-sink-error.md) | When the sink returns an error, the engine MUST continue the run. | approved |
| [REQ-1067](REQ-1067-broken-sink-keeps-run-alive.md) | Rendering is not the work, so a broken sink MUST NOT destroy a run in progress. | approved |
| [REQ-1068](REQ-1068-broken-sink-not-silent.md) | A broken sink MUST NOT fail silently. | approved |
| [REQ-1069](REQ-1069-engine-works-without-sink.md) | The engine MUST function with no sink attached. | approved |
| [REQ-1070](REQ-1070-parsed-value-present-when-finished.md) | When a spec declares an output schema, the outcome's parsed value MUST be present when the stop reason is `finished`. | approved |
| [REQ-1071](REQ-1071-parsed-value-absent-otherwise.md) | When a spec declares an output schema, the outcome's parsed value MUST be absent when the stop reason is not `finished`. | approved |
| [REQ-1072](REQ-1072-invalid-final-response-retried.md) | A final response that fails schema validation MUST be retried per [R-LLM-051] before the run stops with `failed`. | approved |
| [REQ-1073](REQ-1073-request-ceilings-are-declarable.md) | A declaration on a provider or a model MUST accept a ceiling of requests per minute and a ceiling of requests per day. | approved |
| [REQ-1074](REQ-1074-request-count-shared-across-invocations.md) | Requests MUST be counted in the workspace database, so that two invocations share one ceiling. A limit that resets with the process isn't a limit: run `meow` in a loop and every invocation starts with a full allowance. A test that runs two processes and shows the second refused by what the first spent is what tells a shared limiter from a per-process one. | approved |
| [REQ-1075](REQ-1075-over-ceiling-request-refused-before-call.md) | A request that would exceed a declared ceiling MUST be refused before the call is made. | approved |
| [REQ-1076](REQ-1076-ceiling-refusal-names-limit-and-release.md) | The refusal of a request over a ceiling MUST name which limit it hit and when that limit frees up. | approved |
| [REQ-1077](REQ-1077-cost-cap-needs-a-price-source.md) | A budget's cost axis MUST NOT be declarable unless something supplies the price of the calls it bounds. A user who caps cost and gets a cap that silently does nothing is worse off than one who was never offered it. | approved |
| [REQ-1078](REQ-1078-cost-cap-needs-priced-model.md) | A workspace MUST fail to load when a budget caps cost for an agent whose model declares no price. | approved |
| [REQ-1200](REQ-1200-credential-resolution-order.md) | A credential MUST be resolved in this order: the `api_key` the declaration gives, then the global store, then the provider's environment variable. The first that is present and not empty wins. | approved |
| [REQ-1201](REQ-1201-missing-credential-names-three-places.md) | A provider with no credential from any of the three MUST fail naming all three places, with the environment variable spelled out, so the reader can act without consulting a document. | approved |
| [REQ-1202](REQ-1202-no-resolution-during-declaration.md) | Resolution MUST NOT happen while `.meow/` is being evaluated beyond what `[R-STAR-084]` already allows: a declaration may call `env.get`, and the store is read when a provider is built rather than when it is declared. | approved |
| [REQ-1203](REQ-1203-no-module-exposes-credential-store.md) | A runtime module MUST NOT expose the credential store. A handler that wants a secret reads it from the environment, which is a decision the person running the command made. | approved |
| [REQ-1204](REQ-1204-credentials-live-in-auth-json.md) | Credentials MUST live in one file at `~/.meow/auth.json`, outside every workspace, so that a repository cannot carry one and a contributor cannot commit one. | superseded |
| [REQ-1205](REQ-1205-store-created-owner-only.md) | The credential store file MUST be created with owner-only permissions. | approved |
| [REQ-1206](REQ-1206-auth-refuses-readable-store.md) | `meow auth` MUST refuse to read a credential store file that is readable by anyone else, naming the file and the permission it expects. | approved |
| [REQ-1207](REQ-1207-store-write-is-atomic.md) | Writing the store MUST be atomic. | approved |
| [REQ-1208](REQ-1208-store-write-never-truncates.md) | A write of the store MUST NOT leave a truncated file if the process dies, because a truncated credential store locks the user out of every provider at once. | approved |
| [REQ-1209](REQ-1209-auth-login-stores-credential.md) | `meow auth login <provider>` MUST store a credential. | approved |
| [REQ-1210](REQ-1210-auth-logout-removes-credential.md) | `meow auth logout <provider>` MUST remove a credential. | approved |
| [REQ-1211](REQ-1211-auth-list-shows-providers-only.md) | `meow auth list` MUST show which providers have a credential without showing any of them. | approved |
| [REQ-1212](REQ-1212-no-command-prints-credential.md) | A command MUST NOT print a credential, in full or in part. A listing says that a credential is present and when it was stored. | approved |
| [REQ-1213](REQ-1213-oauth-uses-device-code-flow.md) | A provider kind that authenticates by OAuth rather than an API key MUST obtain its credential through a device-code flow: the command shows a code and a URL, waits for the person to approve, and stores the result. | approved |
| [REQ-1214](REQ-1214-device-flow-honours-interval.md) | The device-code flow MUST honour the interval the authorisation server asks for. | approved |
| [REQ-1215](REQ-1215-device-flow-stops-on-expiry.md) | The device-code flow MUST stop when the server says the code expired. | approved |
| [REQ-1216](REQ-1216-device-flow-is-interruptible.md) | The device-code flow MUST be interruptible, so that Ctrl-C leaves nothing half-written. | approved |
| [REQ-1217](REQ-1217-refresh-token-in-same-store.md) | A refresh token MUST be kept in the same store as an API key. | approved |
| [REQ-1218](REQ-1218-expired-token-refreshed-silently.md) | An access token that has expired MUST be refreshed without asking the person again. | approved |
| [REQ-1219](REQ-1219-failed-refresh-says-so.md) | A refresh that fails MUST say so. | approved |
| [REQ-1220](REQ-1220-failed-refresh-names-command.md) | A refresh that fails MUST say which command re-authenticates. | approved |
| [REQ-1221](REQ-1221-oauth-login-needs-terminal.md) | `meow auth login` for an OAuth provider MUST be refused when there is no terminal, because there is nobody to read the code. | approved |
| [REQ-1222](REQ-1222-untrusted-workspace-shows-declarations.md) | The first invocation in a workspace this machine has not agreed to MUST show what the workspace declares - its agents, its tools, and the policy it asks for. | approved |
| [REQ-1223](REQ-1223-untrusted-workspace-asks-once.md) | The first invocation in a workspace this machine has not agreed to MUST ask once. | approved |
| [REQ-1224](REQ-1224-untrusted-without-terminal-fails.md) | A run in an untrusted workspace with no terminal MUST fail rather than proceed or block. | approved |
| [REQ-1225](REQ-1225-untrusted-failure-names-trust-command.md) | A run in an untrusted workspace with no terminal MUST name the command that grants trust. | approved |
| [REQ-1226](REQ-1226-trust-recorded-per-workspace-path.md) | Trust MUST be recorded per workspace path in `~/.meow/trust.json`. | approved |
| [REQ-1227](REQ-1227-trust-is-withdrawable.md) | Trust MUST be withdrawable. | approved |
| [REQ-1228](REQ-1228-trust-prompt-runs-only-declaration.md) | Loading `.meow/` to ask the question MUST NOT run any of it beyond declaration, which `[R-STAR-084]` already guarantees: the prompt describes what was declared, and declaring reaches nothing. | approved |
| [REQ-1229](REQ-1229-changed-declarations-lose-trust.md) | A workspace whose declarations changed MUST NOT silently keep its trust. What is recorded is what was shown, so a `.meow/` that grows a new tool or widens its policy asks again. | approved |
| [REQ-1230](REQ-1230-credentials-kept-in-platform-secret-store.md) | Credentials MUST be kept in the platform secret store: Keychain on macOS, Credential Manager on Windows, and Secret Service (`libsecret`) on Linux. meowg1k persists nothing itself and asks the operating system to, because OAuth forces something to persist and the principle forbids meowg1k being the thing that holds the secret. | approved |
| [REQ-1231](REQ-1231-credential-held-only-for-the-call.md) | A credential MUST be held in memory only for the duration of the call that uses it. | approved |
| [REQ-1232](REQ-1232-auth-json-is-the-fallback-store.md) | Where no platform secret store is running, credentials MUST fall back to `~/.meow/auth.json`. | approved |
| [REQ-1233](REQ-1233-fallback-store-use-is-announced.md) | The credential store MUST say so when it uses the fallback file `~/.meow/auth.json`. | approved |
| [REQ-1400](REQ-1400-indexing-respects-ignore-files.md) | Indexing MUST respect `.gitignore`, `.meowignore`, and the workspace's own `.meow/.data/` exclusion. | approved |
| [REQ-1401](REQ-1401-meowignore-supports-negation.md) | `.meowignore` MUST support negation, so a path `.gitignore` excludes can be indexed deliberately. | approved |
| [REQ-1402](REQ-1402-meow-data-never-reincludable.md) | `.meow/.data/` MUST NOT be re-includable. | approved |
| [REQ-1403](REQ-1403-indexing-skips-binary-files.md) | Indexing MUST skip files it detects as binary. | approved |
| [REQ-1404](REQ-1404-indexing-counts-binary-skips.md) | Indexing MUST record how many files it skipped as binary. | approved |
| [REQ-1405](REQ-1405-indexing-skips-oversized-files.md) | Indexing MUST skip a file larger than a configured size. | approved |
| [REQ-1406](REQ-1406-indexing-reports-each-size-skip.md) | Indexing MUST report each skip of a file larger than the configured size, so a missing result is explainable. | approved |
| [REQ-1407](REQ-1407-no-symlink-leaving-workspace.md) | Indexing MUST NOT follow a symlink that leaves the workspace. | approved |
| [REQ-1408](REQ-1408-prose-indexed-like-code.md) | Indexing MUST NOT exclude a file for being prose rather than source. Markdown and plain text are indexed on the same terms as code, subject to the same exclusions. | approved |
| [REQ-1409](REQ-1409-chunking-is-deterministic.md) | Chunking MUST be deterministic. | approved |
| [REQ-1410](REQ-1410-same-content-same-chunks.md) | The same file content MUST produce the same chunks, with the same boundaries, on every run. | approved |
| [REQ-1411](REQ-1411-chunk-carries-path-and-ranges.md) | A chunk MUST carry its source path, its byte range, and its line range, so a result can be cited precisely. | approved |
| [REQ-1412](REQ-1412-chunks-overlap-by-lines.md) | Chunks MUST overlap by a configured number of lines, so a definition split across a boundary is retrievable from either side. Overlap is counted in lines rather than tokens because boundaries are lines, and the two units cannot both be exact. | approved |
| [REQ-1413](REQ-1413-chunk-within-embedding-input-limit.md) | A chunk MUST NOT exceed the embedding model's input limit. | approved |
| [REQ-1414](REQ-1414-boundaries-fall-on-lines.md) | Chunk boundaries MUST fall on line boundaries. | approved |
| [REQ-1415](REQ-1415-chunk-never-splits-a-line.md) | A chunk MUST NOT begin or end part-way through a line, except where a single line exceeds the embedding model's input limit. | approved |
| [REQ-1416](REQ-1416-overlong-line-is-split.md) | Where a single line exceeds the embedding model's input limit, that line MUST be split, so a minified file cannot make the two rules unsatisfiable. | approved |
| [REQ-1417](REQ-1417-line-split-is-reported.md) | Where a single line exceeds the embedding model's input limit and is split, the split MUST be reported. | approved |
| [REQ-1418](REQ-1418-chunks-embedded-in-batches.md) | Chunks MUST be embedded in batches. | approved |
| [REQ-1419](REQ-1419-rejected-batch-split-and-retried.md) | A batch rejected for exceeding a provider limit MUST be split and retried. | approved |
| [REQ-1420](REQ-1420-oversized-chunk-error-names-file.md) | A single chunk that a provider rejects as too large MUST fail with an error naming the file and the chunk's line range, not a generic size error. | approved |
| [REQ-1421](REQ-1421-embedding-is-resumable.md) | Embedding MUST be resumable. | approved |
| [REQ-1422](REQ-1422-interrupted-build-keeps-stored-vectors.md) | An interrupted build MUST NOT re-embed chunks whose vectors are already stored. | approved |
| [REQ-1423](REQ-1423-update-reembeds-only-changed-files.md) | Update MUST re-embed a file only when its content hash differs from the stored one. | approved |
| [REQ-1424](REQ-1424-hash-covers-chunking-parameters.md) | The content hash MUST cover the chunking parameters as well as the content, so changing chunk size or overlap does not leave stale chunks that look current. | approved |
| [REQ-1425](REQ-1425-update-removes-gone-files.md) | Update MUST remove the chunks of a file that no longer exists or that has become excluded. | approved |
| [REQ-1426](REQ-1426-update-reports-file-counts.md) | Update MUST report how many files were added, changed, removed, and unchanged. | approved |
| [REQ-1427](REQ-1427-query-ranked-by-similarity.md) | A query MUST return results ranked by descending similarity, each carrying the path, the line range, the chunk text, and the score. | approved |
| [REQ-1428](REQ-1428-empty-index-query-says-empty.md) | A query against an empty or absent index MUST return no results and say the index is empty. | approved |
| [REQ-1429](REQ-1429-query-never-builds-index.md) | A query against an empty or absent index MUST NOT build one implicitly. | approved |
| [REQ-1430](REQ-1430-query-accepts-limit-and-score.md) | A query MUST accept a result limit and a minimum score. | approved |
| [REQ-1431](REQ-1431-query-applies-limit-and-score.md) | A query MUST apply both its result limit and its minimum score. | approved |
| [REQ-1432](REQ-1432-cached-query-needs-no-network.md) | A query MUST be answerable without a network call when the embedding of the query itself is cached. | approved |
| [REQ-1433](REQ-1433-query-accepts-glob-path-filter.md) | A query MUST accept a path filter, expressed as globs. | approved |
| [REQ-1434](REQ-1434-path-filter-applied-before-ranking.md) | A query MUST apply its path filter before ranking. | approved |
| [REQ-1435](REQ-1435-vectors-share-one-database.md) | Vectors MUST be stored through `meow-store` in the same database as everything else. | approved |
| [REQ-1436](REQ-1436-index-records-embedding-model.md) | The index MUST record which embedding model produced it. | approved |
| [REQ-1437](REQ-1437-model-mismatch-query-fails.md) | A query against an index built by a different embedding model MUST fail with both names rather than returning meaningless scores. | approved |
| [REQ-1438](REQ-1438-clear-removes-vectors-and-chunks.md) | `clear` MUST remove every vector and chunk. | approved |
| [REQ-1439](REQ-1439-clear-keeps-sessions-and-store.md) | `clear` MUST leave sessions and the key-value store untouched. | approved |
| [REQ-1600](REQ-1600-provider-implements-generation.md) | A provider MUST implement generation. | approved |
| [REQ-1601](REQ-1601-provider-declares-its-capabilities.md) | A provider MUST declare whether it supports streaming, tool calling, and embeddings, and whether its structured output is native or emulated. | approved |
| [REQ-1602](REQ-1602-undeclared-capability-fails-before-sending.md) | Calling a capability a provider does not declare MUST fail with an error naming the provider and the capability, before any request is sent. | approved |
| [REQ-1603](REQ-1603-no-vendor-type-in-trait.md) | A trait method signature MUST NOT name a type from a vendor SDK or wire format. | approved |
| [REQ-1604](REQ-1604-credential-obtained-per-request.md) | What a provider sends to authenticate MUST be obtained per request rather than fixed when the provider is built, so that a credential which expires can be renewed without rebuilding anything. | approved |
| [REQ-1605](REQ-1605-renew-credential-once-per-lifetime.md) | Renewing a provider's credential MUST happen at most once for a credential's lifetime. | approved |
| [REQ-1606](REQ-1606-failed-renewal-names-reauth-command.md) | A credential renewal that fails MUST fail the request naming the provider and the command that re-authenticates, per [R-AUTH-022]. | approved |
| [REQ-1607](REQ-1607-message-carries-one-role.md) | A message MUST carry exactly one role: system, user, assistant, or tool. | approved |
| [REQ-1608](REQ-1608-assistant-message-carries-text-and-calls.md) | An assistant message MUST be able to carry both text and a list of tool calls in the same message. | approved |
| [REQ-1609](REQ-1609-tool-result-carries-call-identifier.md) | A tool result message MUST carry the identifier of the tool call it answers. | approved |
| [REQ-1610](REQ-1610-provider-assigns-missing-call-identifiers.md) | A provider MUST assign an identifier to every tool call it returns that lacks one. | approved |
| [REQ-1611](REQ-1611-call-identifiers-unique-per-response.md) | The identifiers of the tool calls a provider returns MUST be unique within a response. | approved |
| [REQ-1612](REQ-1612-provider-deduplicates-repeated-tool-calls.md) | A provider MUST deduplicate tool calls that arrive more than once with the same identifier, before returning the response, regardless of any session or history setting. | approved |
| [REQ-1613](REQ-1613-message-may-carry-cache-hint.md) | A message MAY carry a cache hint. | approved |
| [REQ-1614](REQ-1614-explicit-breakpoint-provider-translates-hint.md) | A provider whose API has explicit cache breakpoints MUST translate a message's cache hint into one. | approved |
| [REQ-1615](REQ-1615-automatic-cache-provider-ignores-hint.md) | A provider that caches automatically MUST ignore a message's cache hint. | approved |
| [REQ-1616](REQ-1616-cache-hint-keeps-message-content.md) | A message's cache hint MUST NOT change the content of the message. | approved |
| [REQ-1617](REQ-1617-stream-event-kinds-are-fixed.md) | The stream event kinds MUST be exactly: `Text`, `Thinking`, `ToolCallStart`, `ToolCallDelta`, `ToolCallEnd`, `Usage`, `Error`, and `Done`. | approved |
| [REQ-1618](REQ-1618-aggregated-stream-matches-body.md) | Aggregating a recorded stream MUST produce the same response value as parsing the non-streaming body for the same model output. | approved |
| [REQ-1619](REQ-1619-no-synthesised-stream-events.md) | A provider that has no native streaming endpoint MUST declare streaming as unsupported rather than synthesising events from a completed response. | approved |
| [REQ-1620](REQ-1620-consumer-error-aborts-request.md) | An error raised by the stream consumer MUST abort the request. | approved |
| [REQ-1621](REQ-1621-consumer-error-propagates-unchanged.md) | An error raised by the stream consumer MUST propagate to the caller unchanged. | approved |
| [REQ-1622](REQ-1622-thinking-delivered-as-events.md) | Thinking content MUST be delivered as `Thinking` stream events. | approved |
| [REQ-1623](REQ-1623-thinking-preserved-on-message.md) | Thinking content MUST be preserved on the assistant message it belongs to, because a provider can require it to be sent back on a later turn that continues a tool call. | approved |
| [REQ-1624](REQ-1624-errors-classified-into-three-kinds.md) | Every provider error MUST be classified as exactly one of `Transient`, `Fatal`, or `QuotaExhausted`. | approved |
| [REQ-1625](REQ-1625-transient-status-codes-and-timeouts.md) | HTTP 408, 429, 500, 502, 503, 504, and connection or read timeouts MUST classify as `Transient`. | approved |
| [REQ-1626](REQ-1626-fatal-status-codes-and-schema-rejections.md) | HTTP 400, 401, 403, 404, and 422, and any schema rejection, MUST classify as `Fatal`. | approved |
| [REQ-1627](REQ-1627-only-transient-errors-retried.md) | Only `Transient` errors MUST be retried. | approved |
| [REQ-1628](REQ-1628-fatal-error-surfaces-first-time.md) | A `Fatal` error MUST surface to the caller on the first occurrence, without delay. | approved |
| [REQ-1629](REQ-1629-retry-uses-jittered-exponential-backoff.md) | Retry MUST use exponential backoff with random jitter. | approved |
| [REQ-1630](REQ-1630-retry-honours-retry-after.md) | Retry MUST honour a `Retry-After` header when the response carries one. | approved |
| [REQ-1631](REQ-1631-retry-stops-at-maximum-attempts.md) | Retry MUST stop after a configured maximum attempt count. | approved |
| [REQ-1632](REQ-1632-quota-exhausted-surfaces-immediately.md) | A `QuotaExhausted` error MUST surface immediately with the provider named. | approved |
| [REQ-1633](REQ-1633-quota-exhausted-not-retried.md) | A `QuotaExhausted` error MUST NOT be retried. | approved |
| [REQ-1634](REQ-1634-quota-told-by-documented-signal.md) | A provider MUST distinguish `QuotaExhausted` from a rate limit using its own documented signal, not by matching text in a message. | approved |
| [REQ-1635](REQ-1635-unsignalled-429-is-transient.md) | When a provider's documented quota signal is absent, a 429 MUST classify as `Transient`, because retrying a spent quota costs a delay while refusing a rate limit costs the run. | approved |
| [REQ-1636](REQ-1636-retry-checks-cancellation-token.md) | Every retry MUST check the cancellation token before sleeping and before the next attempt. | approved |
| [REQ-1637](REQ-1637-response-reports-prompt-and-completion.md) | A response MUST report prompt tokens and completion tokens. | approved |
| [REQ-1638](REQ-1638-response-reports-cached-prompt-tokens.md) | A response MUST report cached prompt tokens as a separate optional count. | approved |
| [REQ-1639](REQ-1639-unreported-cache-count-left-absent.md) | A provider that does not report caching MUST leave the cached prompt token count absent, never zero, so that "no cache hits" stays distinguishable from "this provider does not say". | approved |
| [REQ-1640](REQ-1640-missing-usage-stated-explicitly.md) | When a provider reports no usage at all, the response MUST say so explicitly, so that a missing count is distinguishable from a zero count. | approved |
| [REQ-1641](REQ-1641-request-may-carry-json-schema.md) | A request MAY carry a JSON Schema. | approved |
| [REQ-1642](REQ-1642-native-structured-output-uses-schema.md) | A provider that declares native structured output MUST use a request's JSON Schema. | approved |
| [REQ-1643](REQ-1643-emulated-structured-output-validates-json.md) | A provider that declares emulated structured output MUST request JSON and validate the result against the schema. | approved |
| [REQ-1644](REQ-1644-schema-failure-retried-with-error.md) | A response that fails schema validation MUST be retried with the validation error included in the follow-up request, up to a configured attempt count. | approved |
| [REQ-1645](REQ-1645-schema-failure-ends-with-last-error.md) | A response that fails schema validation MUST, once the configured attempt count is spent, fail with the last validation error. | approved |
| [REQ-1646](REQ-1646-valid-response-returned-parsed.md) | A response that satisfies the schema MUST be returned parsed, not as a string. | approved |
| [REQ-1647](REQ-1647-request-takes-cancellation-token.md) | Every request MUST take a cancellation token. | approved |
| [REQ-1648](REQ-1648-cancellation-aborts-inflight-http.md) | Every request MUST abort the in-flight HTTP request when its cancellation token fires. | approved |
| [REQ-1649](REQ-1649-cancellation-returns-distinct-error.md) | A cancelled request MUST return a distinct cancellation error, never a timeout or a transport error. | approved |
| [REQ-1650](REQ-1650-transport-trusts-native-roots.md) | The provider transport MUST verify a vendor's certificate against the operating system's root certificate store. | approved |
| [REQ-1651](REQ-1651-failed-stream-fails-at-the-call.md) | A streaming request that a vendor answers with a non-success status MUST fail at the call, never through the stream the call would have returned. | approved |
| [REQ-1652](REQ-1652-connect-gives-up-after-30-seconds.md) | The provider transport MUST give up connecting to a vendor after 30 seconds. | approved |
| [REQ-1653](REQ-1653-no-deadline-on-a-whole-response.md) | The provider transport MUST NOT put a deadline on a whole response, because a model thinking for two minutes is normal. | approved |
| [REQ-1800](REQ-1800-workspace-declares-every-package.md) | A workspace MUST declare every package it loads, with a name, a source, and a version. (from docs/spec/packages.md [R-PKG-001], high) | approved |
| [REQ-1801](REQ-1801-undeclared-package-load-fails.md) | `load("@<pkg>//<path>", ...)` naming an undeclared package MUST fail saying so. (from docs/spec/packages.md [R-PKG-001], high) | approved |
| [REQ-1802](REQ-1802-undeclared-package-load-fetches-nothing.md) | `load("@<pkg>//<path>", ...)` naming an undeclared package MUST NOT fetch anything. (from docs/spec/packages.md [R-PKG-001], high) | approved |
| [REQ-1803](REQ-1803-duplicate-package-name-refused.md) | A declaration MUST be refused at load time when two packages claim the same name, naming both. (from docs/spec/packages.md [R-PKG-002], high) | approved |
| [REQ-1804](REQ-1804-package-name-never-std.md) | A package name MUST NOT be `std`. The scheme that reaches the runtime modules cannot be shadowed by something fetched. (from docs/spec/packages.md [R-PKG-003], high) | approved |
| [REQ-1805](REQ-1805-lockfile-pins-version-and-hash.md) | Every package MUST be pinned in `.meow/meow.lock` by its resolved version and the SHA-256 of its contents. (from docs/spec/packages.md [R-PKG-010], high) | approved |
| [REQ-1806](REQ-1806-lockfile-is-deterministic.md) | The lockfile MUST be deterministic: the same declarations and the same upstream produce the same bytes, so a diff shows a dependency change and nothing else. (from docs/spec/packages.md [R-PKG-010], high) | approved |
| [REQ-1807](REQ-1807-load-verifies-hash-against-lockfile.md) | Loading MUST verify the hash of what it is about to run against the lockfile. (from docs/spec/packages.md [R-PKG-011], high) | approved |
| [REQ-1808](REQ-1808-hash-mismatch-fails-before-evaluation.md) | Loading MUST fail without evaluating anything when the hash of what it is about to run and the lockfile differ, naming the package, the expected hash, and the one found. (from docs/spec/packages.md [R-PKG-011], high) | approved |
| [REQ-1809](REQ-1809-unlocked-package-names-lock-command.md) | Loading MUST fail when a declared package is absent from the lockfile, naming the command that writes one. (from docs/spec/packages.md [R-PKG-012], high) | approved |
| [REQ-1810](REQ-1810-load-never-fetches-silently.md) | Loading MUST NOT fetch silently: a load that reaches the network without being asked is a load that can change behaviour between two runs of the same commit. (from docs/spec/packages.md [R-PKG-012], high) | approved |
| [REQ-1811](REQ-1811-pkg-update-rewrites-lockfile.md) | `meow pkg update` MUST re-resolve versions and rewrite the lockfile. (from docs/spec/packages.md [R-PKG-013], high) | approved |
| [REQ-1812](REQ-1812-pkg-fetch-downloads-pinned-packages.md) | `meow pkg fetch` MUST download what the lockfile already pins. (from docs/spec/packages.md [R-PKG-013], high) | approved |
| [REQ-1813](REQ-1813-pkg-fetch-leaves-lockfile-unchanged.md) | `meow pkg fetch` MUST NOT change the lockfile. (from docs/spec/packages.md [R-PKG-013], high) | approved |
| [REQ-1814](REQ-1814-cached-package-loads-offline.md) | A package whose contents are already in the cache and match the lockfile MUST load without any network access. (from docs/spec/packages.md [R-PKG-020], high) | approved |
| [REQ-1815](REQ-1815-fetched-workspace-works-offline.md) | A workspace that has fetched once MUST work offline. (from docs/spec/packages.md [R-PKG-020], high) | approved |
| [REQ-1816](REQ-1816-cache-keyed-by-hash.md) | The cache MUST be keyed by hash, so two workspaces pinning the same package share one copy and a changed pin cannot be served the old bytes. (from docs/spec/packages.md [R-PKG-021], high) | approved |
| [REQ-1817](REQ-1817-fetch-is-atomic.md) | A fetch MUST be atomic. (from docs/spec/packages.md [R-PKG-022], high) | approved |
| [REQ-1818](REQ-1818-interrupted-download-leaves-no-package.md) | An interrupted download MUST NOT leave something the next run mistakes for a complete package. (from docs/spec/packages.md [R-PKG-022], high) | approved |
| [REQ-1819](REQ-1819-fetch-carries-deadline.md) | A fetch MUST carry a deadline. (from docs/spec/packages.md [R-PKG-023], high) | approved |
| [REQ-1820](REQ-1820-fetch-is-interruptible.md) | A fetch MUST be interruptible. (from docs/spec/packages.md [R-PKG-023], high) | approved |
| [REQ-1821](REQ-1821-package-loads-std-and-siblings.md) | A file in a package MUST be able to `load("@std//...")` and to load another file in the same package by a relative path. (from docs/spec/packages.md [R-PKG-030], high) | approved |
| [REQ-1822](REQ-1822-package-cannot-load-host-workspace.md) | A file in a package MUST NOT be able to load `//<path>`, which is the host workspace's own tree. A dependency reaching into the workspace that depends on it inverts the direction and makes the package's behaviour depend on who loaded it. (from docs/spec/packages.md [R-PKG-031], high) | approved |
| [REQ-1823](REQ-1823-package-cannot-load-undeclared-package.md) | A package MUST NOT be able to load another package the host workspace did not declare. Dependencies are the workspace's to state, so that `meow.lock` is the whole list. (from docs/spec/packages.md [R-PKG-032], high) | approved |
| [REQ-1824](REQ-1824-package-code-follows-workspace-rules.md) | Code from a package MUST run under the same rules as code the workspace wrote: the declaration phase reaches nothing, and what a model decides still passes through policy. A package is Starlark, not a plugin. (from docs/spec/packages.md [R-PKG-033], high) | approved |
| [REQ-1825](REQ-1825-source-is-gzipped-tarball-over-http.md) | A package source MUST be a gzipped tar archive fetched over HTTP or HTTPS. | approved |
| [REQ-1826](REQ-1826-fetch-refuses-escaping-entry-name.md) | A fetch MUST refuse an archive holding an entry whose path contains `..` or starts at a root. | approved |
| [REQ-1827](REQ-1827-fetch-fails-on-declined-entry.md) | A fetch MUST fail when the unpacker declines to write an entry, and never pin a package without it. | approved |
| [REQ-1828](REQ-1828-archive-over-64-mib-refused.md) | A fetch MUST refuse an archive larger than 64 MiB. | approved |
| [REQ-1829](REQ-1829-fetch-gives-up-after-120-seconds.md) | A fetch MUST give up when the download hasn't finished within 120 seconds. | approved |
| [REQ-2000](REQ-2000-rule-matches-tool-name-pattern.md) | A rule MUST match on a tool name pattern. | approved |
| [REQ-2001](REQ-2001-rule-may-narrow-with-selector.md) | A rule MAY narrow further with a selector belonging to that tool: `paths` for file tools, `commands` for shell tools. | approved |
| [REQ-2002](REQ-2002-pattern-supports-trailing-wildcard-exact-name.md) | A tool name pattern MUST support a trailing wildcard (`fs.*`) and an exact name (`fs.read`). | approved |
| [REQ-2003](REQ-2003-pattern-rejects-leading-wildcard.md) | A tool name pattern MUST NOT support a leading wildcard. | approved |
| [REQ-2004](REQ-2004-paths-selector-matches-resolved-path.md) | A `paths` selector MUST match against an absolute path with symlinks already resolved, using glob semantics where `` crosses directory boundaries. | approved |
| [REQ-2005](REQ-2005-caller-resolves-path-before-evaluation.md) | Resolving the path is the caller's job and MUST happen before evaluation, so that evaluation itself touches no filesystem. | approved |
| [REQ-2006](REQ-2006-tool-acts-on-evaluated-path.md) | A tool MUST act on the exact path the policy evaluated. | approved |
| [REQ-2007](REQ-2007-tool-does-not-reresolve-path.md) | A tool MUST NOT resolve the path a second time. Re-resolving reopens the window in which a path allowed as a file becomes a symlink to somewhere denied. | approved |
| [REQ-2008](REQ-2008-commands-selector-matches-command-line.md) | A `commands` selector MUST match against the full command line as a single string, using glob semantics. | approved |
| [REQ-2009](REQ-2009-unsupported-selector-fails-at-build.md) | A selector that no tool matching the rule's name pattern supports MUST fail when the policy is built, not when a call is evaluated. | approved |
| [REQ-2010](REQ-2010-multi-path-call-evaluated-per-path.md) | A call that touches several paths MUST be evaluated once per resolved path. | approved |
| [REQ-2011](REQ-2011-partial-read-returns-allowed-paths.md) | A read that hits a denied path MUST return the allowed paths. | approved |
| [REQ-2012](REQ-2012-partial-read-names-skipped-paths.md) | A read that hits a denied path MUST name every path it skipped, so the model knows its view is partial. | approved |
| [REQ-2013](REQ-2013-write-to-denied-path-denied-whole.md) | A write that hits a denied path MUST be denied as a whole. | approved |
| [REQ-2014](REQ-2014-denied-write-writes-no-path.md) | A write that hits a denied path MUST NOT write any path, so the workspace is never left half-applied. | approved |
| [REQ-2015](REQ-2015-network-tools-support-hosts-selector.md) | Network tools MUST support a `hosts` selector matching the host of the request, so a policy can allow one host without allowing the network. | approved |
| [REQ-2016](REQ-2016-evaluation-checks-deny-ask-allow.md) | Evaluation MUST check deny rules first, then ask rules, then allow rules. | approved |
| [REQ-2017](REQ-2017-evaluation-returns-first-match.md) | Evaluation MUST return the first match. | approved |
| [REQ-2018](REQ-2018-unmatched-call-is-denied.md) | A call that matches no rule MUST be denied. | approved |
| [REQ-2019](REQ-2019-decision-names-its-rule.md) | A decision MUST name the rule that produced it, or record that no rule matched. | approved |
| [REQ-2020](REQ-2020-evaluation-touches-no-io.md) | Evaluation MUST touch no filesystem, network, or clock. | approved |
| [REQ-2021](REQ-2021-evaluation-is-deterministic.md) | Evaluation MUST return the same decision for the same resolved call, the same policy, and the same set of session grants. Grants are an input to evaluation, not a side effect of it. | approved |
| [REQ-2022](REQ-2022-ask-denies-without-terminal.md) | An `ask` decision MUST resolve to `deny` when standard input is not a terminal, or when the invocation set `--yes`. | approved |
| [REQ-2023](REQ-2023-prompt-shows-exact-arguments.md) | An approval prompt MUST show the tool name and the exact arguments the call would use, verbatim. | approved |
| [REQ-2024](REQ-2024-prompt-shows-no-paraphrase.md) | An approval prompt MUST NOT show a summary or a paraphrase. | approved |
| [REQ-2025](REQ-2025-prompt-names-causing-rule.md) | An approval prompt MUST name the rule that caused it. | approved |
| [REQ-2026](REQ-2026-always-grant-lasts-one-process.md) | An approval granted as "always" MUST apply for the current process only. | approved |
| [REQ-2027](REQ-2027-policy-writes-no-grant-file.md) | The policy layer MUST NOT write a grant back to any file. | approved |
| [REQ-2028](REQ-2028-prompt-waits-indefinitely-by-default.md) | An approval prompt MUST wait indefinitely by default. | approved |
| [REQ-2029](REQ-2029-prompt-timeout-may-be-configured.md) | A timeout on an approval prompt MAY be configured. | approved |
| [REQ-2030](REQ-2030-expired-prompt-resolves-to-deny.md) | When a timeout is configured and expires, the approval prompt MUST resolve to `deny`. | approved |
| [REQ-2031](REQ-2031-agent-may-declare-own-policy.md) | An agent MAY declare a policy of its own. | approved |
| [REQ-2032](REQ-2032-effective-decision-is-more-restrictive.md) | For every call, the effective decision MUST be the more restrictive of what the workspace policy and the agent policy say, ordering `deny` above `ask` above `allow`. | approved |
| [REQ-2033](REQ-2033-agent-policy-cannot-allow-denied.md) | An agent policy MUST NOT allow a call the workspace policy denies. | approved |
| [REQ-2034](REQ-2034-agent-policy-cannot-relax-ask.md) | An agent policy MUST NOT turn an `ask` into an `allow`. | approved |
| [REQ-2035](REQ-2035-subagent-inherits-caller-policy.md) | A sub-agent MUST inherit its caller's effective policy. | approved |
| [REQ-2036](REQ-2036-subagent-may-narrow-policy.md) | A sub-agent MAY narrow its caller's effective policy further, by the same rule. | approved |
| [REQ-2037](REQ-2037-engine-evaluates-before-execution.md) | The engine MUST evaluate the policy before the tool executes. | approved |
| [REQ-2038](REQ-2038-engine-skips-denied-tool.md) | The engine MUST NOT execute a tool whose decision is `deny`. | approved |
| [REQ-2039](REQ-2039-denied-call-tells-the-model.md) | A denied call MUST return a message to the model naming the tool and stating that policy denied it, so the model can choose another approach. | approved |
| [REQ-2040](REQ-2040-every-evaluation-emits-policy-event.md) | Every evaluation MUST produce a `Policy` session event, whatever the decision. | approved |
| [REQ-2041](REQ-2041-rule-may-mark-value-sensitive.md) | A rule MAY mark an argument or a result field sensitive. | approved |
| [REQ-2042](REQ-2042-sensitive-value-redacted-everywhere.md) | A value marked sensitive MUST be redacted in the approval prompt, in the transcript, and in every export. | approved |
| [REQ-2043](REQ-2043-sensitive-value-never-logged-clear.md) | A value marked sensitive MUST NOT be written to the session log in the clear. | approved |
| [REQ-2044](REQ-2044-redaction-uses-fixed-placeholder.md) | Redaction MUST replace the value with a fixed placeholder. | approved |
| [REQ-2045](REQ-2045-redaction-hides-value-length.md) | Redaction MUST NOT reveal the length of the value. | approved |
| [REQ-2046](REQ-2046-explain-returns-rule-decision.md) | `meow policy explain <tool> <argument>` MUST return the rule decision a real call with those arguments would receive: `allow`, `ask`, or `deny`. | approved |
| [REQ-2047](REQ-2047-explain-does-not-predict-answers.md) | `meow policy explain` MUST NOT claim to predict how an `ask` would be answered, because that depends on a person and on grants made during a run that has not happened. | approved |
| [REQ-2048](REQ-2048-explain-names-rule-and-location.md) | The explanation MUST name the matching rule and the file and line it was declared on. | approved |
| [REQ-2049](REQ-2049-explain-counts-checked-rules.md) | The explanation MUST report how many higher-precedence rules were checked without matching. | approved |
| [REQ-2050](REQ-2050-path-arguments-reach-policy-as-paths.md) | A tool call MUST reach the policy with the values of its arguments named `path`, `file`, `paths` and `files` as the paths it touches, each resolved against the workspace root. | approved |
| [REQ-2051](REQ-2051-command-argument-reaches-policy.md) | A tool call MUST reach the policy with the value of its argument named `command` as its command line. | approved |
| [REQ-2052](REQ-2052-url-argument-reaches-policy-as-host.md) | A tool call MUST reach the policy with the host of its argument named `url` as the host it reaches. | approved |
| [REQ-2053](REQ-2053-write-access-read-from-tool-name.md) | A tool call MUST reach the policy as a write when the tool's name contains `write`, `remove`, `append` or `mkdir`, and as a read otherwise. | approved |
| [REQ-2200](REQ-2200-event-log-is-append-only.md) | The event log MUST be append-only. | approved |
| [REQ-2201](REQ-2201-written-events-never-changed.md) | The session layer MUST NOT update or delete an event that has been written. | approved |
| [REQ-2202](REQ-2202-sequence-numbers-monotonic-gapless.md) | Each event MUST carry a sequence number that is monotonic and gapless within its session, starting at 1. | approved |
| [REQ-2203](REQ-2203-event-kinds-are-fixed.md) | The event kinds MUST be exactly: `Started`, `UserMessage`, `Assistant`, `ToolCall`, `ToolResult`, `Policy`, `Usage`, `Compaction`, `Note`, and `Finished`. | approved |
| [REQ-2204](REQ-2204-events-carry-utc-timestamp.md) | Every event MUST carry the timestamp at which it was recorded, in UTC. | approved |
| [REQ-2205](REQ-2205-session-holds-one-or-more-runs.md) | A session MUST hold one or more runs. | approved |
| [REQ-2206](REQ-2206-run-begins-with-started.md) | Each run MUST begin with a `Started` event. | approved |
| [REQ-2207](REQ-2207-ended-run-closed-by-finished.md) | Once it has ended, each run MUST be closed by exactly one `Finished` event before the next `Started` event. | approved |
| [REQ-2208](REQ-2208-resume-appends-started-event.md) | Resuming a session MUST append a new `Started` event, so that a session that has already ended can be continued without rewriting the event that ended it. | approved |
| [REQ-2209](REQ-2209-compaction-names-superseded-range.md) | A `Compaction` event MUST name the inclusive sequence range it supersedes. | approved |
| [REQ-2210](REQ-2210-compaction-keeps-superseded-events.md) | A `Compaction` event MUST NOT delete the events in the range it supersedes. | approved |
| [REQ-2211](REQ-2211-model-rebuild-substitutes-summary.md) | Rebuilding the message list for a model call MUST skip every superseded range and substitute that range's summary. | approved |
| [REQ-2212](REQ-2212-display-rebuild-returns-originals.md) | Rebuilding the log for display or export MUST return the original events. | approved |
| [REQ-2213](REQ-2213-display-rebuild-has-no-summary.md) | Rebuilding the log for display or export MUST NOT substitute a summary. | approved |
| [REQ-2214](REQ-2214-compactions-do-not-overlap.md) | A `Compaction` event MUST NOT supersede a range that an earlier `Compaction` event already supersedes. | approved |
| [REQ-2215](REQ-2215-usage-event-has-typed-fields.md) | A `Usage` event MUST carry prompt tokens, completion tokens, cached prompt tokens, and cost as separate typed fields. | approved |
| [REQ-2216](REQ-2216-cost-computed-at-write-time.md) | Cost MUST be computed when the event is written, from the price table in effect at that moment. | approved |
| [REQ-2217](REQ-2217-cost-not-recomputed-on-read.md) | Cost MUST NOT be recomputed on read. | approved |
| [REQ-2218](REQ-2218-unpriced-model-records-absent-cost.md) | A model with no price in the table MUST record the cost as absent, never as zero, so an unpriced model does not read as a free one. | approved |
| [REQ-2219](REQ-2219-totals-include-child-sessions.md) | A session MUST report its totals as the sum of its own `Usage` events plus the totals of its child sessions. | approved |
| [REQ-2220](REQ-2220-session-has-full-and-short-identifier.md) | Every session MUST have a full identifier that sorts by creation time, and a short identifier that is its last eight characters. The tail comes from the identifier's entropy part, the clock's sub-millisecond digits and the process id, so it distinguishes sessions the timestamp cannot (from crates/meow-cli/src/wire.rs, high). The length is fixed rather than "shortest unique" so that an identifier written into a commit message keeps resolving as sessions accumulate. | approved |
| [REQ-2221](REQ-2221-ambiguous-short-id-lists-candidates.md) | Resolving a short identifier that matches more than one session MUST fail with an error listing the candidates. | approved |
| [REQ-2222](REQ-2222-ambiguous-short-id-picks-none.md) | Resolving a short identifier that matches more than one session MUST NOT pick one. | approved |
| [REQ-2223](REQ-2223-selectors-resolve-at-use.md) | The selectors `@last`, `@last-N`, and `@<agent-name>` MUST resolve at the moment of use: `@last` to the most recent session in the workspace, `@last-N` to the Nth most recent, and `@<agent-name>` to the most recent session of that agent. | approved |
| [REQ-2224](REQ-2224-session-may-carry-name.md) | A session MAY carry a name. | approved |
| [REQ-2225](REQ-2225-session-name-unique-in-workspace.md) | A session name MUST be unique within the workspace. | approved |
| [REQ-2226](REQ-2226-session-name-avoids-at-sign.md) | A session name MUST NOT begin with `@`. | approved |
| [REQ-2227](REQ-2227-session-name-resolves-like-identifier.md) | A session name MUST resolve like an identifier, so that a name can never shadow a selector. | approved |
| [REQ-2228](REQ-2228-session-has-exactly-one-state.md) | A session MUST be in exactly one state: `running`, or one of the terminal states `finished`, `budget`, `cancelled`, `denied`, `tool_aborted`, or `failed`. | approved |
| [REQ-2229](REQ-2229-log-defines-session-state.md) | The state MUST be defined by the log: `running` when the last event is not `Finished`, and otherwise the stop reason that last `Finished` event carries. | approved |
| [REQ-2230](REQ-2230-state-copy-may-be-denormalised.md) | A denormalised copy of the session state MAY be kept so that listing a thousand sessions does not read a thousand events, provided it is rebuildable from the log and the log wins on any disagreement. | approved |
| [REQ-2231](REQ-2231-writer-updates-heartbeat-at-interval.md) | While a run is in flight its writer MUST update a heartbeat timestamp on the session row at a fixed interval. | approved |
| [REQ-2232](REQ-2232-heartbeat-is-not-an-event.md) | The heartbeat MUST NOT be an event, because it is mutable and the log is not. | approved |
| [REQ-2233](REQ-2233-stale-heartbeat-means-dead-session.md) | A session whose last event is not `Finished` and whose heartbeat is older than three intervals MUST be treated as dead. | approved |
| [REQ-2234](REQ-2234-dead-session-gets-failed-finish.md) | Opening a session whose last event is not `Finished` and whose recording process is no longer alive MUST append `Finished { stop: failed, reason: "process exited" }`. | approved |
| [REQ-2235](REQ-2235-dead-session-not-reported-running.md) | Opening a session whose last event is not `Finished` and whose recording process is no longer alive MUST NOT leave the session reported as running. | approved |
| [REQ-2236](REQ-2236-resume-continues-existing-sequence.md) | Resuming a session MUST append to it, continuing the existing sequence. | approved |
| [REQ-2237](REQ-2237-resume-creates-no-new-session.md) | Resuming a session MUST NOT create a new session. | approved |
| [REQ-2238](REQ-2238-resume-rebuilds-with-compaction.md) | Resuming MUST rebuild the message list per [R-SESSION-011], so a resumed run sees compaction exactly as the original did. | approved |
| [REQ-2239](REQ-2239-fork-copies-first-n-events.md) | Forking at sequence `n` MUST create a new session whose first `n` events are copies referencing the same blobs. | approved |
| [REQ-2240](REQ-2240-fork-increments-blob-reference-counts.md) | Forking at sequence `n` MUST increment the reference count of every blob so referenced. Without the increment, collecting the origin would delete content the fork still points at. | approved |
| [REQ-2241](REQ-2241-fork-records-its-origin.md) | Forking at sequence `n` MUST record the origin session and sequence. | approved |
| [REQ-2242](REQ-2242-fork-leaves-origin-unchanged.md) | Forking at sequence `n` MUST NOT modify the origin. | approved |
| [REQ-2243](REQ-2243-invalid-fork-point-names-range.md) | Forking at a sequence that does not exist, or at a sequence inside a superseded range, MUST fail with an error naming the valid range. | approved |
| [REQ-2244](REQ-2244-unnamed-run-starts-fresh-session.md) | A run that names no session MUST start a fresh one. | approved |
| [REQ-2245](REQ-2245-no-continuation-inferred-from-workspace.md) | The session layer MUST NOT infer continuation from the workspace. | approved |
| [REQ-2246](REQ-2246-subagent-session-records-parent.md) | A session started by a sub-agent MUST record its parent. | approved |
| [REQ-2247](REQ-2247-parent-enumerates-children-in-order.md) | The parent of a sub-agent session MUST be able to enumerate its children in creation order. | approved |
| [REQ-2248](REQ-2248-parent-relation-has-no-cycle.md) | A cycle in the parent relation MUST be impossible. | approved |
| [REQ-2249](REQ-2249-parent-exists-before-child.md) | A session's parent MUST already exist when the session is created. | approved |
| [REQ-2250](REQ-2250-policy-event-precedes-tool-run.md) | Every tool invocation MUST produce a `Policy` event before the tool runs, recording the decision, the rule that produced it, and whether the decision came from a rule, an interactive answer, or a session grant. | approved |
| [REQ-2251](REQ-2251-policy-event-for-denied-calls.md) | A `Policy` event MUST be written for denied invocations as well as allowed ones. | approved |
| [REQ-2252](REQ-2252-gc-deletes-whole-sessions-only.md) | Garbage collection MUST delete whole sessions only. | approved |
| [REQ-2253](REQ-2253-gc-deletes-parent-with-descendants.md) | Garbage collection MUST delete a parent only together with its descendants. | approved |
| [REQ-2254](REQ-2254-gc-keeps-named-sessions.md) | Garbage collection MUST NOT delete a named session unless the caller asks for named sessions explicitly. | approved |
| [REQ-2255](REQ-2255-retention-configurable-by-three-limits.md) | Retention MUST be configurable by age, by count, and by total database size. | approved |
| [REQ-2256](REQ-2256-retention-applies-strictest-limit.md) | Retention MUST apply the strictest of the configured limits. | approved |
| [REQ-2257](REQ-2257-json-export-uses-live-schema.md) | JSON export MUST use the schema definition and version number that the live `--format json` renderer uses. | approved |
| [REQ-2258](REQ-2258-event-kinds-serialise-identically.md) | Every persisted event kind MUST serialise identically in JSON export and in the live `--format json` renderer, per [R-TUI-032]. | approved |
| [REQ-2259](REQ-2259-markdown-export-contents.md) | Markdown export MUST include the transcript, the tool calls with their policy decisions, and the usage totals. | approved |
| [REQ-2260](REQ-2260-export-redacts-sensitive-values.md) | Export MUST redact every value the policy marked sensitive under [R-POLICY-060], in both formats. | approved |
| [REQ-2261](REQ-2261-export-omits-thinking-by-default.md) | Export MUST omit thinking content unless it is asked for explicitly, because it is the part of a transcript least likely to be meant for an audience. | approved |
| [REQ-2262](REQ-2262-priced-call-records-its-cost.md) | A call against a model that declares a price MUST record its cost from the usage the provider returned: prompt tokens at the input price plus completion tokens at the output price, each price per million tokens. | approved |
| [REQ-2263](REQ-2263-no-v02-session-store-read.md) | The binary MUST NOT read or migrate a v0.2.x session store, which v0.2.x kept at `.meowg1k/.data/project.db`. | approved |
| [REQ-2264](REQ-2264-tool-result-may-omit-output.md) | A `ToolResult` event MAY carry an empty output, recording the call's identifier, duration and error without what the tool returned. | approved |
| [REQ-2400](REQ-2400-workspace-root-nearest-ancestor.md) | The workspace root MUST be the nearest ancestor directory of the working directory containing `.meow/meow.star`. (from docs/spec/starlark.md [R-STAR-001], high) | approved |
| [REQ-2401](REQ-2401-discovery-stops-at-first-match.md) | Discovery MUST stop at the first match. (from docs/spec/starlark.md [R-STAR-001], high) | approved |
| [REQ-2402](REQ-2402-discovery-never-merges-global-configuration.md) | Discovery MUST NOT merge with a global configuration. (from docs/spec/starlark.md [R-STAR-001], high) | approved |
| [REQ-2403](REQ-2403-outside-workspace-names-searched-directories.md) | Running outside any workspace MUST fail with an error naming the directories searched. (from docs/spec/starlark.md [R-STAR-002], high) | approved |
| [REQ-2404](REQ-2404-outside-workspace-suggests-meow-init.md) | Running outside any workspace MUST suggest `meow init`. (from docs/spec/starlark.md [R-STAR-002], high) | approved |
| [REQ-2405](REQ-2405-std-load-resolves-runtime-module.md) | `load("@std//<name>", ...)` MUST resolve to a runtime module. (from docs/spec/starlark.md [R-STAR-003], high) | approved |
| [REQ-2406](REQ-2406-unknown-std-module-lists-available.md) | In `load("@std//<name>", ...)`, a name with no module MUST fail listing the available modules. (from docs/spec/starlark.md [R-STAR-003], high) | approved |
| [REQ-2407](REQ-2407-local-load-resolves-under-meow.md) | `load("//<path>", ...)` MUST resolve relative to `.meow/`. (from docs/spec/starlark.md [R-STAR-004], high) | approved |
| [REQ-2408](REQ-2408-local-load-refuses-escaping-path.md) | `load("//<path>", ...)` MUST fail for a path that escapes `.meow/`. (from docs/spec/starlark.md [R-STAR-004], high) | approved |
| [REQ-2409](REQ-2409-package-load-resolves-declared-package.md) | `load("@<pkg>//<path>", ...)` MUST resolve to a declared package, per `docs/spec/packages.md`. (from docs/spec/starlark.md [R-STAR-005], high) | approved |
| [REQ-2410](REQ-2410-package-name-never-local-path.md) | `load("@<pkg>//<path>", ...)` MUST NOT be interpreted as a local path. (from docs/spec/starlark.md [R-STAR-005], high) | approved |
| [REQ-2411](REQ-2411-undeclared-package-name-fails.md) | In `load("@<pkg>//<path>", ...)`, a name no declaration gives MUST fail saying so. (from docs/spec/starlark.md [R-STAR-005], high) | approved |
| [REQ-2412](REQ-2412-load-detects-import-cycle.md) | Loading MUST detect an import cycle and fail with the cycle listed in order. (from docs/spec/starlark.md [R-STAR-006], high) | approved |
| [REQ-2413](REQ-2413-file-evaluated-once-per-invocation.md) | A file MUST be evaluated at most once per invocation, whatever the number of `load` statements naming it. (from docs/spec/starlark.md [R-STAR-007], high) | approved |
| [REQ-2414](REQ-2414-runtime-modules-in-one-table.md) | Runtime modules MUST be registered in one table. (from docs/spec/starlark.md [R-STAR-010], high) | approved |
| [REQ-2415](REQ-2415-consumers-obtain-modules-from-table.md) | Every consumer MUST obtain a module from the one table runtime modules are registered in. No code path may construct a context or module set of its own. (from docs/spec/starlark.md [R-STAR-010], high) | approved |
| [REQ-2416](REQ-2416-agent-loop-handler-same-modules.md) | A tool handler invoked inside an agent loop MUST see exactly the same modules, with the same behaviour, as a handler invoked from the command line. (from docs/spec/starlark.md [R-STAR-011], high) | approved |
| [REQ-2417](REQ-2417-table-contains-re-and-time.md) | The table MUST contain `re` and `time`. (from docs/spec/starlark.md [R-STAR-012], high) | approved |
| [REQ-2418](REQ-2418-re-time-identical-in-agent-loop.md) | `re` and `time` each MUST behave identically in a handler invoked from the command line and in one invoked inside an agent loop. (from docs/spec/starlark.md [R-STAR-012], high) | approved |
| [REQ-2419](REQ-2419-re-calls-accept-pattern-subject.md) | `re.match`, `re.find_all`, `re.replace`, and `re.split` MUST accept a pattern and a subject. (from docs/spec/starlark.md [R-STAR-013], high) | approved |
| [REQ-2420](REQ-2420-re-rejects-uncompilable-pattern.md) | `re.match`, `re.find_all`, `re.replace`, and `re.split` MUST fail at the call with the pattern and the reason when the pattern does not compile. (from docs/spec/starlark.md [R-STAR-013], high) | approved |
| [REQ-2421](REQ-2421-re-no-match-not-error.md) | A pattern that compiles but matches nothing MUST NOT be an error. (from docs/spec/starlark.md [R-STAR-013], high) | approved |
| [REQ-2422](REQ-2422-re-match-returns-none-unmatched.md) | For a pattern that compiles but matches nothing, `match` MUST return `None`. (from docs/spec/starlark.md [R-STAR-013], high) | approved |
| [REQ-2423](REQ-2423-re-find-all-split-return-list.md) | For a pattern that compiles but matches nothing, `find_all` and `split` MUST return a list. (from docs/spec/starlark.md [R-STAR-013], high) | approved |
| [REQ-2424](REQ-2424-time-works-in-utc-seconds.md) | `time.now`, `time.parse`, `time.format`, and `time.since` MUST work in a single scale - a UTC instant counted in seconds. (from docs/spec/starlark.md [R-STAR-014], high) | approved |
| [REQ-2425](REQ-2425-time-refuses-local-time.md) | `time.now`, `time.parse`, `time.format`, and `time.since` MUST NOT accept or return a local time. (from docs/spec/starlark.md [R-STAR-014], high) | approved |
| [REQ-2426](REQ-2426-duration-independent-of-machine-zone.md) | A handler that reports a duration MUST get the same number whatever the machine's zone. (from docs/spec/starlark.md [R-STAR-014], high) | approved |
| [REQ-2427](REQ-2427-table-contains-data-formats.md) | The table MUST contain `yaml`, `toml`, `csv`, and `xml`. (from docs/spec/starlark.md [R-STAR-015], high) | approved |
| [REQ-2428](REQ-2428-data-formats-expose-parse-encode.md) | `yaml`, `toml`, `csv`, and `xml` each MUST expose `parse` and `encode`. (from docs/spec/starlark.md [R-STAR-015], high) | approved |
| [REQ-2429](REQ-2429-rejected-text-fails-with-reason.md) | Text that the format rejects MUST fail at the call with the reason. (from docs/spec/starlark.md [R-STAR-015], high) | approved |
| [REQ-2430](REQ-2430-rejected-text-never-partial-value.md) | Text that the format rejects MUST NOT be returned as a partial value. (from docs/spec/starlark.md [R-STAR-015], high) | approved |
| [REQ-2431](REQ-2431-parse-yields-same-value-across-formats.md) | `yaml.parse`, `toml.parse`, and `json.parse` MUST produce the same Starlark value for documents that describe the same data. (from docs/spec/starlark.md [R-STAR-016], high) | approved |
| [REQ-2432](REQ-2432-encode-accepts-other-formats-values.md) | Each `encode` of `yaml`, `toml`, and `json` MUST accept any value the other two produce that its own format can represent. TOML has no null, so `toml.encode` is the one encoder that meets a value it can't represent (from https://github.com/meowshed/meowg1k/pull/141, high). (from docs/spec/starlark.md [R-STAR-016], high) | approved |
| [REQ-2433](REQ-2433-handler-converts-formats-blind.md) | A handler MUST be able to read one format and write another without knowing which it read. (from docs/spec/starlark.md [R-STAR-016], high) | approved |
| [REQ-2434](REQ-2434-csv-parse-shape-follows-header.md) | `csv.parse` MUST return a list of dictionaries when the first record names the columns and a list of lists when it does not. (from docs/spec/starlark.md [R-STAR-017], high) | approved |
| [REQ-2435](REQ-2435-csv-parse-names-ragged-record.md) | `csv.parse` MUST fail naming the record number when a record's length disagrees with the header. (from docs/spec/starlark.md [R-STAR-017], high) | approved |
| [REQ-2436](REQ-2436-csv-encode-accepts-either-shape.md) | `csv.encode` MUST accept either shape. (from docs/spec/starlark.md [R-STAR-017], high) | approved |
| [REQ-2437](REQ-2437-xml-parse-returns-element-tree.md) | `xml.parse` MUST return a tree in which every element carries its `tag`, its `attrs`, its `children`, and its `text`. (from docs/spec/starlark.md [R-STAR-018], high) | approved |
| [REQ-2438](REQ-2438-xml-parse-never-flattens-element.md) | `xml.parse` MUST NOT flatten an element into a dictionary. (from docs/spec/starlark.md [R-STAR-018], high) | approved |
| [REQ-2439](REQ-2439-xml-encode-accepts-tree-shapes.md) | `xml.encode` MUST accept that tree, and equally a tree of dictionaries carrying the same four keys. (from docs/spec/starlark.md [R-STAR-018], high) | approved |
| [REQ-2440](REQ-2440-xml-encode-escapes-text-attributes.md) | `xml.encode` MUST escape text and attribute values. (from docs/spec/starlark.md [R-STAR-018], high) | approved |
| [REQ-2441](REQ-2441-search-works-without-index.md) | `search.text` and `search.files` MUST search the workspace without an index. (from docs/spec/starlark.md [R-STAR-019], high) | approved |
| [REQ-2442](REQ-2442-search-obeys-index-walk.md) | `search.text` and `search.files` MUST obey the same walk the index obeys, so a file the index ignores is a file they do not report. (from docs/spec/starlark.md [R-STAR-019], high) | approved |
| [REQ-2443](REQ-2443-search-text-literal-by-default.md) | `search.text` MUST take a literal by default and a regular expression when asked. (from docs/spec/starlark.md [R-STAR-019], high) | approved |
| [REQ-2444](REQ-2444-search-text-reports-path-line.md) | `search.text` MUST report the path and the line number of every hit. (from docs/spec/starlark.md [R-STAR-019], high) | approved |
| [REQ-2445](REQ-2445-handler-context-six-members.md) | The handler context MUST expose exactly six members: `args`, `session`, `out`, `ask`, `stdin`, and `workspace`, plus `cancelled()`. (from docs/spec/starlark.md [R-STAR-020], high) | approved |
| [REQ-2446](REQ-2446-capabilities-not-context-members.md) | Runtime capabilities MUST NOT be members of the context. A handler reaches them by `load`. (from docs/spec/starlark.md [R-STAR-021], high) | approved |
| [REQ-2447](REQ-2447-table-contains-http-module.md) | The table MUST contain `http`, exposing `get`, `post`, `put`, and `delete`. (from docs/spec/starlark.md [R-STAR-022], high) | approved |
| [REQ-2448](REQ-2448-http-call-carries-deadline.md) | Every `http` call MUST carry a deadline. (from docs/spec/starlark.md [R-STAR-022], high) | approved |
| [REQ-2449](REQ-2449-http-call-fails-past-deadline.md) | Every `http` call MUST fail when its deadline passes. (from docs/spec/starlark.md [R-STAR-022], high) | approved |
| [REQ-2450](REQ-2450-http-call-caps-response-read.md) | Every `http` call MUST cap how much of a response it will read. (from docs/spec/starlark.md [R-STAR-022], high) | approved |
| [REQ-2451](REQ-2451-http-cancel-leaves-no-request.md) | Every `http` call MUST NOT leave a request in flight when the run is cancelled. (from docs/spec/starlark.md [R-STAR-022], high) | approved |
| [REQ-2452](REQ-2452-http-returns-any-server-response.md) | A response the server sent MUST be returned rather than raised, whatever its status. (from docs/spec/starlark.md [R-STAR-023], high) | approved |
| [REQ-2453](REQ-2453-http-gives-status-headers-body.md) | An `http` call MUST give the status, the headers, and the body. (from docs/spec/starlark.md [R-STAR-023], high) | approved |
| [REQ-2454](REQ-2454-http-404-not-failure.md) | An `http` call MUST NOT turn a 404 into a failure. (from docs/spec/starlark.md [R-STAR-023], high) | approved |
| [REQ-2455](REQ-2455-unreached-response-fails-with-cause.md) | A call that never reached a response - a name that does not resolve, a refused connection, a deadline - MUST fail with what went wrong. (from docs/spec/starlark.md [R-STAR-023], high) | approved |
| [REQ-2456](REQ-2456-table-contains-index-module.md) | The table MUST contain `index`, exposing `build`, `update`, `stats`, and `query`. (from docs/spec/starlark.md [R-STAR-024], high) | approved |
| [REQ-2457](REQ-2457-index-build-update-report-counts.md) | `build` and `update` MUST report what changed as counts rather than as text. (from docs/spec/starlark.md [R-STAR-024], high) | approved |
| [REQ-2458](REQ-2458-index-query-takes-score-floor.md) | `query` MUST take the floor below which a hit is not worth returning. (from docs/spec/starlark.md [R-STAR-024], high) | approved |
| [REQ-2459](REQ-2459-index-call-without-declaration-fails.md) | An `index` call in a workspace that declares no index MUST fail saying so. (from docs/spec/starlark.md [R-STAR-025], high) | approved |
| [REQ-2460](REQ-2460-index-call-never-chooses-model.md) | An `index` call in a workspace that declares no index MUST NOT choose a model on the workspace's behalf. (from docs/spec/starlark.md [R-STAR-025], high) | approved |
| [REQ-2461](REQ-2461-unbuilt-index-query-fails.md) | A `query` against an index that was never built MUST fail saying to build it. (from docs/spec/starlark.md [R-STAR-025], high) | approved |
| [REQ-2462](REQ-2462-unbuilt-index-query-no-empty-list.md) | A `query` against an index that was never built MUST NOT return an empty list, because no results and no index are different facts. (from docs/spec/starlark.md [R-STAR-025], high) | approved |
| [REQ-2463](REQ-2463-table-contains-store-module.md) | The table MUST contain `store`, exposing `get`, `put`, `delete`, and `keys`. (from docs/spec/starlark.md [R-STAR-026], high) | approved |
| [REQ-2464](REQ-2464-store-value-survives-run.md) | A `store` value MUST survive the run that wrote it. (from docs/spec/starlark.md [R-STAR-026], high) | approved |
| [REQ-2465](REQ-2465-store-value-survives-session-collection.md) | A `store` value MUST survive collecting every session in the workspace. (from docs/spec/starlark.md [R-STAR-026], high) | approved |
| [REQ-2466](REQ-2466-store-get-returns-put-value.md) | `store.get` MUST return the value that `store.put` was given, of the same type, for any value a handler can build. (from docs/spec/starlark.md [R-STAR-027], high) | approved |
| [REQ-2467](REQ-2467-unwritten-key-gives-caller-default.md) | A key that was never written MUST give the caller's default, and `None` when it named none, so that "absent" and "stored `None`" are the same answer only when the caller asked for that. (from docs/spec/starlark.md [R-STAR-027], high) | approved |
| [REQ-2468](REQ-2468-store-delete-reports-key-presence.md) | `store.delete` MUST say whether the key was there. (from docs/spec/starlark.md [R-STAR-028], high) | approved |
| [REQ-2469](REQ-2469-store-delete-absent-key-succeeds.md) | `store.delete` MUST NOT fail for a key that was not there. (from docs/spec/starlark.md [R-STAR-028], high) | approved |
| [REQ-2470](REQ-2470-store-keys-returns-sorted.md) | `store.keys` MUST return the keys sorted. (from docs/spec/starlark.md [R-STAR-028], high) | approved |
| [REQ-2471](REQ-2471-store-keys-takes-prefix.md) | `store.keys` MUST take a prefix. (from docs/spec/starlark.md [R-STAR-028], high) | approved |
| [REQ-2472](REQ-2472-path-speaks-forward-slash.md) | `path` MUST speak one separator, `/`, whatever the platform. (from docs/spec/starlark.md [R-STAR-029], high) | approved |
| [REQ-2473](REQ-2473-path-accepts-backslash-input.md) | `path` MUST accept `\` in what it is given. (from docs/spec/starlark.md [R-STAR-029], high) | approved |
| [REQ-2474](REQ-2474-path-never-returns-backslash.md) | `path` MUST NOT return `\`, so that a path `path.join` built and a path `search.files` reported can be compared, and a `.meow/` written once behaves the same everywhere. (from docs/spec/starlark.md [R-STAR-029], high) | approved |
| [REQ-2475](REQ-2475-declarations-callable-only-during-evaluation.md) | `meow.provider`, `meow.model`, `meow.agent`, `meow.tool`, `meow.command`, and `meow.policy` MUST be callable only while `.meow/` is being evaluated. (from docs/spec/starlark.md [R-STAR-030], high) | approved |
| [REQ-2476](REQ-2476-declarations-fail-inside-handler.md) | `meow.provider`, `meow.model`, `meow.agent`, `meow.tool`, `meow.command`, and `meow.policy` MUST fail inside a handler. (from docs/spec/starlark.md [R-STAR-030], high) | approved |
| [REQ-2477](REQ-2477-duplicate-declaration-names-both-sites.md) | Declaring two providers, models, agents, or tools with the same name MUST fail naming both declaration sites. (from docs/spec/starlark.md [R-STAR-031], high) | approved |
| [REQ-2478](REQ-2478-dangling-reference-fails-at-load.md) | A declaration that names a provider, model, or tool that does not exist MUST fail at load time, not at first use. (from docs/spec/starlark.md [R-STAR-032], high) | approved |
| [REQ-2479](REQ-2479-references-resolved-after-all-declarations.md) | A reference by name MUST be resolved after every declaration file has been evaluated, so that the order of the declarations it names, inside and between files, does not matter. A call that takes a value, such as `meow.command`, needs that value bound first, as any Starlark name does (from https://github.com/meowshed/meowg1k/pull/123 and crates/meow-star/src/declare.rs, high). (from docs/spec/starlark.md [R-STAR-032], high) | approved |
| [REQ-2480](REQ-2480-builtin-collision-fails-at-load.md) | A command whose name collides with a built-in MUST fail at load time naming the collision. (from docs/spec/starlark.md [R-STAR-033], high) | approved |
| [REQ-2481](REQ-2481-command-never-shadows-builtin.md) | A command whose name collides with a built-in MUST NOT shadow the built-in. (from docs/spec/starlark.md [R-STAR-033], high) | approved |
| [REQ-2482](REQ-2482-model-accepts-chat-or-embedding.md) | `meow.model` MUST accept a `kind` of `chat` or `embedding`, defaulting to `chat`. (from docs/spec/starlark.md [R-STAR-034], high) | approved |
| [REQ-2483](REQ-2483-model-kind-mismatch-fails-at-load.md) | An agent naming an embedding model, or an index naming a chat model, MUST fail at load time naming both the model and the kind it is. (from docs/spec/starlark.md [R-STAR-034], high) | approved |
| [REQ-2484](REQ-2484-index-declares-embedding-model.md) | `meow.index` MUST declare which model embeds the workspace. (from docs/spec/starlark.md [R-STAR-035], high) | approved |
| [REQ-2485](REQ-2485-index-sets-chunking-and-limits.md) | `meow.index` MAY set the chunk size, the overlap, and the file-size limit that [R-INDEX-003] and [R-INDEX-012] call configured. (from docs/spec/starlark.md [R-STAR-035], high) | approved |
| [REQ-2486](REQ-2486-index-declared-at-most-once.md) | `meow.index` MUST be declarable at most once. (from docs/spec/starlark.md [R-STAR-035], high) | approved |
| [REQ-2487](REQ-2487-undeclared-index-command-fails.md) | An index command in a workspace that declares no index MUST fail saying so rather than choosing a model. (from docs/spec/starlark.md [R-STAR-035], high) | approved |
| [REQ-2488](REQ-2488-agent-requires-name-model-system.md) | `meow.agent` MUST require `name`, `model`, and `system`. (from docs/spec/starlark.md [R-STAR-040], high) | approved |
| [REQ-2489](REQ-2489-agent-accepts-optional-settings.md) | `meow.agent` MUST accept `about`, `tools`, `budget`, `compaction`, `output`, `on_tool_error`, and `policy`. (from docs/spec/starlark.md [R-STAR-040], high) | approved |
| [REQ-2490](REQ-2490-agent-usable-as-tool.md) | An agent value MUST be usable in another agent's `tools` list. (from docs/spec/starlark.md [R-STAR-041], high) | approved |
| [REQ-2491](REQ-2491-agent-tool-schema-matches-tool.md) | An agent value in another agent's `tools` list MUST present the same schema there as a tool declared with `meow.tool`. (from docs/spec/starlark.md [R-STAR-041], high) | approved |
| [REQ-2492](REQ-2492-agent-run-returns-outcome-fields.md) | `agent.run(task, ...)` MUST return a value carrying `text`, `value`, `stop`, `detail`, `ok`, `usage`, `steps`, and `session`, with `stop` taking one of the six values in [R-AGENT-002] and `detail` carrying the explanation required by [R-AGENT-004]. (from docs/spec/starlark.md [R-STAR-042], high) | approved |
| [REQ-2493](REQ-2493-agent-call-builds-unrun-invocation.md) | `agent.call(...)` MUST build an invocation without running it. (from docs/spec/starlark.md [R-STAR-043], high) | approved |
| [REQ-2494](REQ-2494-parallel-accepts-invocation-list.md) | `meow.parallel([...])` MUST accept a list of invocations `agent.call(...)` builds. (from docs/spec/starlark.md [R-STAR-043], high) | approved |
| [REQ-2495](REQ-2495-parallel-rejects-non-invocations.md) | `meow.parallel` MUST reject anything that is not an invocation, including a Starlark function, with an error explaining that Starlark values cannot cross a thread boundary. (from docs/spec/starlark.md [R-STAR-044], high) | approved |
| [REQ-2496](REQ-2496-markdown-file-declares-agent.md) | A `.md` file under `.meow/agents/` MUST declare an agent whose frontmatter accepts every keyword argument of `meow.agent` except `system`, plus `include`. (from docs/spec/starlark.md [R-STAR-050], high) | approved |
| [REQ-2497](REQ-2497-frontmatter-carrying-system-fails.md) | The body of a markdown agent supplies `system`, so frontmatter carrying it MUST fail. (from docs/spec/starlark.md [R-STAR-050], high) | approved |
| [REQ-2498](REQ-2498-markdown-starlark-agents-same-value.md) | A markdown agent and a Starlark agent MUST produce the same value. (from docs/spec/starlark.md [R-STAR-051], high) | approved |
| [REQ-2499](REQ-2499-markdown-starlark-agents-indistinguishable.md) | A markdown agent and a Starlark agent MUST be indistinguishable to a caller. (from docs/spec/starlark.md [R-STAR-051], high) | approved |
| [REQ-2500](REQ-2500-bad-frontmatter-fails-with-location.md) | Frontmatter that is not valid YAML, or that carries a key outside the set [R-STAR-050] allows, MUST fail at load time with the file and line. (from docs/spec/starlark.md [R-STAR-052], high) | approved |
| [REQ-2501](REQ-2501-lib-markdown-loadable-as-string.md) | A `.md` file under `.meow/lib/` MUST be loadable as a string. (from docs/spec/starlark.md [R-STAR-053], high) | approved |
| [REQ-2502](REQ-2502-frontmatter-may-carry-include.md) | Frontmatter MAY carry an `include` key naming `.meow/lib/*.md` files. (from docs/spec/starlark.md [R-STAR-054], high) | approved |
| [REQ-2503](REQ-2503-includes-prepended-in-order.md) | The contents of the files an `include` key names MUST be prepended to the system prompt in the order given, separated by a blank line. (from docs/spec/starlark.md [R-STAR-054], high) | approved |
| [REQ-2504](REQ-2504-include-only-markdown-composition.md) | `include` MUST be the only composition a markdown agent has. (from docs/spec/starlark.md [R-STAR-054], high) | approved |
| [REQ-2505](REQ-2505-markdown-agent-no-templating.md) | In a markdown agent, there MUST be no substitution, conditional, or loop. (from docs/spec/starlark.md [R-STAR-054], high) | approved |
| [REQ-2506](REQ-2506-arg-supports-six-types.md) | `meow.arg` MUST support the types `string`, `int`, `float`, `bool`, `enum`, and `list`, each with an optional default and an optional `about`. (from docs/spec/starlark.md [R-STAR-060], high) | approved |
| [REQ-2507](REQ-2507-string-arg-accepts-length-pattern.md) | A `string` argument MUST accept `max_len` and `pattern`. (from docs/spec/starlark.md [R-STAR-060], high) | approved |
| [REQ-2508](REQ-2508-numeric-arg-accepts-bounds.md) | `int` and `float` arguments MUST accept `min` and `max`. (from docs/spec/starlark.md [R-STAR-060], high) | approved |
| [REQ-2509](REQ-2509-enum-arg-requires-values.md) | An `enum` argument MUST require its list of values. (from docs/spec/starlark.md [R-STAR-060], high) | approved |
| [REQ-2510](REQ-2510-list-arg-requires-element-type.md) | A `list` argument MUST require an element type. (from docs/spec/starlark.md [R-STAR-060], high) | approved |
| [REQ-2511](REQ-2511-arg-drives-flag-help-schema.md) | One argument declaration MUST produce the command-line flag, the help text, and the JSON Schema sent to the model, so the three cannot drift. (from docs/spec/starlark.md [R-STAR-061], high) | approved |
| [REQ-2512](REQ-2512-positional-arg-takes-index.md) | An argument marked `positional` MUST take an index. (from docs/spec/starlark.md [R-STAR-062], high) | approved |
| [REQ-2513](REQ-2513-positional-indices-unique-contiguous.md) | Positional indices within one tool MUST be unique and contiguous from zero. (from docs/spec/starlark.md [R-STAR-062], high) | approved |
| [REQ-2514](REQ-2514-constraints-enforced-for-model-arguments.md) | A declared constraint MUST be enforced on the command line and on a model-supplied argument alike. (from docs/spec/starlark.md [R-STAR-063], high) | approved |
| [REQ-2515](REQ-2515-schema-builds-seven-kinds.md) | `meow.schema` MUST build object, list, string, int, float, bool, and enum schemas. (from docs/spec/starlark.md [R-STAR-070], high) | approved |
| [REQ-2516](REQ-2516-schema-emits-valid-json-schema.md) | `meow.schema` MUST emit valid JSON Schema. (from docs/spec/starlark.md [R-STAR-070], high) | approved |
| [REQ-2517](REQ-2517-schema-undeclared-required-field-fails.md) | A schema naming a required field it does not declare MUST fail when the schema is built. (from docs/spec/starlark.md [R-STAR-071], high) | approved |
| [REQ-2518](REQ-2518-invocation-owns-one-evaluator.md) | Each invocation MUST own one evaluator. (from docs/spec/starlark.md [R-STAR-080], high) | approved |
| [REQ-2519](REQ-2519-evaluator-stays-on-thread.md) | The runtime MUST NOT move an evaluator or a Starlark value between threads. (from docs/spec/starlark.md [R-STAR-080], high) | approved |
| [REQ-2520](REQ-2520-io-builtin-blocks-script-thread.md) | A builtin that performs input or output MUST block the calling script thread. (from docs/spec/starlark.md [R-STAR-081], high) | approved |
| [REQ-2521](REQ-2521-io-builtin-exposes-no-future.md) | A builtin that performs input or output MUST NOT expose a future or a callback to Starlark. (from docs/spec/starlark.md [R-STAR-081], high) | approved |
| [REQ-2522](REQ-2522-print-points-to-ctx-out.md) | `print` MUST fail with an error directing the caller to `ctx.out`. (from docs/spec/starlark.md [R-STAR-082], high) | approved |
| [REQ-2523](REQ-2523-declaration-file-has-no-effects.md) | A declaration file MUST NOT write files, run commands, or make network requests. (from docs/spec/starlark.md [R-STAR-083], high) | approved |
| [REQ-2524](REQ-2524-declaration-callable-load-and-env.md) | While `.meow/` is being evaluated, only `load` and `@std//env` MUST be callable. (from docs/spec/starlark.md [R-STAR-084], high) | approved |
| [REQ-2525](REQ-2525-other-modules-refused-during-declaration.md) | While `.meow/` is being evaluated, every other runtime module MUST fail with an error saying it is unavailable during declaration. (from docs/spec/starlark.md [R-STAR-084], high) | approved |
| [REQ-2526](REQ-2526-errors-carry-source-location.md) | Every load-time and run-time error MUST carry the file, line, and column of the Starlark expression that caused it. (from docs/spec/starlark.md [R-STAR-090], high) | approved |
| [REQ-2527](REQ-2527-errors-suggest-closest-name.md) | An error naming an unknown parameter, module, or type MUST suggest the closest declared name when one is within a small edit distance. (from docs/spec/starlark.md [R-STAR-091], high) | approved |
| [REQ-2528](REQ-2528-omitted-budget-axis-takes-default.md) | An agent declaration's `budget` that omits an axis MUST take that axis from the default budget, whether the agent is declared in Starlark or in markdown frontmatter. | approved |
| [REQ-2529](REQ-2529-toml-encode-refuses-null.md) | `toml.encode` MUST fail, saying that TOML has no null, when a value it is given holds `None`, and never drop the key. | approved |
| [REQ-2530](REQ-2530-model-declares-token-prices.md) | `meow.model` MUST accept an input price and an output price per million tokens, as `input_per_mtok` and `output_per_mtok`. | approved |
| [REQ-2531](REQ-2531-meow-on-takes-typed-callbacks.md) | `meow.on` MUST take `text`, `tool` and `step` as separate callbacks, each optional, so a handler observes a run without switching on a kind field. | approved |
| [REQ-2532](REQ-2532-run-without-on-uses-renderer.md) | `agent.run` MUST draw a run with the built-in renderer when it's given no `on`. | approved |
| [REQ-2533](REQ-2533-no-preset-declaration.md) | The `meow` global MUST NOT offer a preset declaration, because a named `meow.model` carrying its own temperature is what a v0.2.x preset named. | approved |
| [REQ-2534](REQ-2534-env-require-fails-at-load.md) | `env.require` MUST fail while the workspace loads, naming the variable, when that variable is unset. | approved |
| [REQ-2535](REQ-2535-table-omits-crypto-template-math.md) | The module table MUST NOT contain `crypto`, `template` or `math`. | approved |
| [REQ-2536](REQ-2536-explicit-none-treated-as-absent.md) | A `meow` declaration builtin MUST treat an explicit `None`, given for an optional keyword argument whose default is absent, as if the argument were left out. | approved |
| [REQ-2537](REQ-2537-fs-remove-refuses-workspace-root.md) | `fs.remove` MUST refuse the workspace root. | approved |
| [REQ-2538](REQ-2538-fs-remove-refuses-the-store.md) | `fs.remove` MUST refuse `.meow/.data/` and every path inside it. | approved |
| [REQ-2539](REQ-2539-fs-follows-links-given-relative-paths.md) | `fs` MAY follow a symbolic link inside the workspace that points outside it, when the call gives a relative path. | approved |
| [REQ-2540](REQ-2540-git-runs-the-users-git.md) | `@std//git` MUST run the user's own `git` program as a subprocess, so it reads the repository with the configuration, hooks and worktrees that program sees. | approved |
| [REQ-2541](REQ-2541-git-refuses-values-starting-with-dash.md) | `@std//git` MUST refuse a revision or a path a caller supplies that begins with `-`. | approved |
| [REQ-2542](REQ-2542-git-commit-takes-only-staged.md) | `git.commit` MUST commit what is staged and nothing else. | approved |
| [REQ-2543](REQ-2543-git-status-returns-porcelain-text.md) | `git.status` MUST return `git`'s porcelain v1 output as text. | approved |
| [REQ-2544](REQ-2544-http-refuses-other-schemes.md) | `http` MUST refuse a URL that doesn't begin with `http://` or `https://`, naming the URL, before it makes any request. | approved |
| [REQ-2545](REQ-2545-search-code-ranks-by-meaning.md) | `search.code` MUST return at most `limit` hits, 10 by default, ranked against the question by meaning. | approved |
| [REQ-2546](REQ-2546-search-code-has-no-score-floor.md) | `search.code` MUST return hits whatever their score, as `index.query` does with a floor of zero. | approved |
| [REQ-2547](REQ-2547-search-code-hit-carries-location.md) | Every hit `search.code` returns MUST carry `path`, `first_line`, `last_line`, `text` and `score`, so a result can be cited and not only read. | approved |
| [REQ-2548](REQ-2548-search-code-narrows-by-path-globs.md) | `search.code` MUST accept `paths`, a list of globs, and return only hits in files those globs match. | approved |
| [REQ-2549](REQ-2549-search-code-without-index-fails.md) | `search.code` MUST fail, saying why, in a workspace with no usable index. | approved |
| [REQ-2550](REQ-2550-fs-glob-returns-sorted-relative-paths.md) | `fs.glob` MUST return, sorted, every file in the workspace whose path relative to the workspace root matches the pattern. | approved |
| [REQ-2551](REQ-2551-fs-glob-skips-data-directories.md) | `fs.glob` MUST NOT walk into a directory named `.data`. | approved |
| [REQ-2552](REQ-2552-fs-glob-rejects-invalid-pattern.md) | `fs.glob` MUST fail, naming the pattern, when the pattern isn't a valid glob. | approved |
| [REQ-2553](REQ-2553-csv-parse-takes-header-argument.md) | `csv.parse` MUST take a `header` argument, `True` by default, saying whether the first record names the columns. | approved |
| [REQ-2600](REQ-2600-one-database-per-workspace.md) | The store MUST keep all data for one workspace in a single SQLite database at `.meow/.data/meow.db`, resolved from the workspace root. | approved |
| [REQ-2601](REQ-2601-wal-and-foreign-keys-enabled.md) | The store MUST enable write-ahead logging and foreign key enforcement on every connection. | approved |
| [REQ-2602](REQ-2602-synchronous-set-to-normal.md) | The store MUST set `synchronous` to `NORMAL`. | approved |
| [REQ-2603](REQ-2603-database-not-encrypted.md) | The store MUST NOT encrypt the database. | approved |
| [REQ-2604](REQ-2604-doctor-reports-unencrypted-database.md) | `meow doctor` MUST report that the database is unencrypted, together with its path, so that a user deciding what a workspace may hold is told rather than left to assume. | approved |
| [REQ-2605](REQ-2605-database-files-owner-only.md) | The database and its side files MUST be created readable and writable by their owner only, on every platform that has file permissions. | approved |
| [REQ-2606](REQ-2606-schema-version-recorded-in-meta.md) | The store MUST record the schema version it was written under in a `meta` table. | approved |
| [REQ-2607](REQ-2607-pending-migrations-applied-in-order.md) | On open, the store MUST apply every migration whose version is greater than the recorded one, in ascending order, each inside its own transaction. | approved |
| [REQ-2608](REQ-2608-failed-migration-keeps-prior-version.md) | A migration that fails MUST leave the database at the version it had before that migration. | approved |
| [REQ-2609](REQ-2609-migrations-are-forward-only.md) | Migrations MUST be forward-only. | approved |
| [REQ-2610](REQ-2610-no-downgrade-path.md) | The store MUST NOT contain a downgrade path. | approved |
| [REQ-2611](REQ-2611-newer-schema-fails-naming-versions.md) | On opening a database whose recorded schema version is greater than the binary supports, the store MUST fail with an error naming both versions. | approved |
| [REQ-2612](REQ-2612-newer-schema-file-left-unmodified.md) | On opening a database whose recorded schema version is greater than the binary supports, the store MUST NOT modify the file. | approved |
| [REQ-2613](REQ-2613-payload-addressed-by-blake3.md) | Every event payload MUST be addressed by the BLAKE3 hash of its content. | approved |
| [REQ-2614](REQ-2614-small-payload-may-be-inline.md) | The store MAY keep a payload under 512 bytes inline rather than in the `blobs` table. | approved |
| [REQ-2615](REQ-2615-inline-storage-invisible-to-reader.md) | The choice to keep a payload inline or in the `blobs` table MUST be invisible to a reader. | approved |
| [REQ-2616](REQ-2616-same-hash-returns-same-bytes.md) | The same hash MUST return the same bytes whether its payload is kept inline or in the `blobs` table. | approved |
| [REQ-2617](REQ-2617-duplicate-blob-stored-once.md) | Writing a blob whose hash is already present MUST NOT store a second copy. | approved |
| [REQ-2618](REQ-2618-duplicate-blob-write-succeeds.md) | Writing a blob whose hash is already present MUST NOT fail. | approved |
| [REQ-2619](REQ-2619-reference-count-per-blob.md) | The store MUST maintain a reference count per blob. | approved |
| [REQ-2620](REQ-2620-blob-deleted-at-zero-references.md) | The store MUST delete a blob only when its reference count reaches zero. | approved |
| [REQ-2621](REQ-2621-missing-blob-fails-naming-hash.md) | Reading a blob whose hash has no row MUST fail with an error naming the hash. | approved |
| [REQ-2622](REQ-2622-missing-blob-never-empty.md) | Reading a blob whose hash has no row MUST NOT return empty content. | approved |
| [REQ-2623](REQ-2623-each-event-committed-alone.md) | Each event MUST be committed on its own. | approved |
| [REQ-2624](REQ-2624-no-transaction-across-tool-execution.md) | A turn MUST NOT be written in one transaction held open across tool execution: the log is append-only, so a half-written turn is a true record of how far the run got, and holding the single write lock for the length of a tool would block every other session and lose that tool's result on a crash. | approved |
| [REQ-2625](REQ-2625-failed-write-propagates-to-caller.md) | A failed write MUST propagate as an error to the caller. | approved |
| [REQ-2626](REQ-2626-no-log-and-continue.md) | The store MUST NOT log a failure and continue. | approved |
| [REQ-2627](REQ-2627-one-writer-at-a-time.md) | The store MUST allow one writer at a time. | approved |
| [REQ-2628](REQ-2628-readers-concurrent-with-writer.md) | The store MUST allow readers concurrently with the writer. | approved |
| [REQ-2629](REQ-2629-bulk-write-commits-in-batches.md) | A bulk write such as an index build MUST commit in bounded batches. | approved |
| [REQ-2630](REQ-2630-bulk-write-releases-write-lock.md) | A bulk write such as an index build MUST NOT hold the write lock for the length of the operation, so that indexing cannot block an agent run. | approved |
| [REQ-2631](REQ-2631-busy-timeout-at-least-five-seconds.md) | A write that blocks on another writer MUST wait up to a configured busy timeout of at least 5 seconds before failing. | approved |
| [REQ-2632](REQ-2632-durable-workspace-key-value-table.md) | The store MUST provide a durable key-value table, scoped to the workspace, with `get`, `put`, `delete`, and `keys` operations. | approved |
| [REQ-2633](REQ-2633-key-value-separate-from-sessions.md) | Workspace key-value data MUST be stored separately from session state, so that deleting a session leaves it intact. | approved |
| [REQ-2634](REQ-2634-session-deletion-decrements-references.md) | The store MUST delete a session by removing its rows and decrementing the reference count of every blob those rows referenced. | approved |
| [REQ-2635](REQ-2635-no-partial-session-deletion.md) | The store MUST NOT delete part of a session. | approved |
| [REQ-2636](REQ-2636-store-reports-database-size.md) | The store MUST provide the total on-disk size of the database so retention can act on it. | approved |
| [REQ-2637](REQ-2637-cache-keyed-by-request-hash.md) | The store MUST provide a cache keyed by a hash of the request that produced the entry. | approved |
| [REQ-2638](REQ-2638-embeddings-cached-by-default.md) | Embedding responses MUST be cached by default. | approved |
| [REQ-2639](REQ-2639-generations-uncached-unless-asked.md) | Generation responses MUST NOT be cached unless the caller asks, because an agent that retries wants a fresh attempt and a cache would hand it the answer that already failed. | approved |
| [REQ-2640](REQ-2640-cache-entry-records-model.md) | A cache entry MUST record the model that produced it. | approved |
| [REQ-2641](REQ-2641-cache-misses-on-other-model.md) | A cache lookup MUST miss when the model differs, so that changing a model cannot return another model's answer. | approved |
| [REQ-2642](REQ-2642-eviction-by-size-and-age.md) | Cache eviction MUST be by total size and age. | approved |
| [REQ-2643](REQ-2643-eviction-spares-quoting-sessions.md) | Evicting a cache entry MUST NOT affect any session that quoted it. | approved |
| [REQ-2800](REQ-2800-renderer-chosen-by-runtime.md) | The renderer MUST be chosen by the runtime, never by a script. | approved |
| [REQ-2801](REQ-2801-format-json-selects-json-renderer.md) | `--format json` MUST select the JSON renderer whatever the terminal is. | approved |
| [REQ-2802](REQ-2802-non-terminal-selects-plain-renderer.md) | A stdout that is not a terminal, or `NO_COLOR` in the environment, or `--color=never`, MUST select the plain renderer. | approved |
| [REQ-2803](REQ-2803-otherwise-inline-renderer-selected.md) | Otherwise the inline terminal renderer MUST be selected. | approved |
| [REQ-2804](REQ-2804-renderers-accept-one-event-stream.md) | All three renderers MUST accept the same event stream. | approved |
| [REQ-2805](REQ-2805-renderers-drivable-from-recorded-log.md) | Each of the three renderers MUST be drivable from a recorded log with no terminal attached. | approved |
| [REQ-2806](REQ-2806-terminal-renderer-uses-inline-viewport.md) | The terminal renderer MUST use an inline viewport. | approved |
| [REQ-2807](REQ-2807-no-alternate-screen-buffer.md) | The terminal renderer MUST NOT switch to the alternate screen buffer. | approved |
| [REQ-2808](REQ-2808-finalized-line-written-to-scrollback.md) | A finalized transcript line MUST be written into scrollback. | approved |
| [REQ-2809](REQ-2809-finalized-line-never-redrawn.md) | A finalized transcript line MUST NOT be redrawn afterwards. | approved |
| [REQ-2810](REQ-2810-only-live-region-redrawn.md) | Only the live region MUST be redrawn. | approved |
| [REQ-2811](REQ-2811-live-region-shows-run-state.md) | The live region MUST show the current tool, the elapsed time, the step count, and the budget consumed. | approved |
| [REQ-2812](REQ-2812-final-line-names-stop-reason.md) | On any exit the process can observe, including an interrupt, the live region MUST be replaced by a final line naming the stop reason. | approved |
| [REQ-2813](REQ-2813-scrollback-valid-after-unobserved-exit.md) | The transcript already in scrollback MUST remain valid even on an exit the process cannot observe, which is what committing finalized lines immediately buys. | approved |
| [REQ-2814](REQ-2814-resize-reflows-only-live-region.md) | A terminal resize MUST reflow only the live region. | approved |
| [REQ-2815](REQ-2815-diagnostics-enter-scrollback-in-order.md) | Diagnostics from the logging layer MUST be inserted into scrollback in order. | approved |
| [REQ-2816](REQ-2816-diagnostics-never-over-live-region.md) | Diagnostics from the logging layer MUST NOT be drawn over the live region. | approved |
| [REQ-2817](REQ-2817-live-region-three-rows-high.md) | The live region MUST be three rows high. | approved |
| [REQ-2818](REQ-2818-live-region-keeps-its-height.md) | The live region MUST NOT change height. | approved |
| [REQ-2819](REQ-2819-approval-prompt-committed-to-transcript.md) | An approval prompt MUST be committed to the transcript rather than drawn in the live region. | approved |
| [REQ-2820](REQ-2820-live-region-says-answer-awaited.md) | While an approval prompt is open the live region MUST say that an answer is awaited. | approved |
| [REQ-2821](REQ-2821-plain-renderer-emits-no-ansi.md) | The plain renderer MUST emit no ANSI escape sequences. | approved |
| [REQ-2822](REQ-2822-plain-renderer-never-moves-cursor.md) | The plain renderer MUST NOT move the cursor. | approved |
| [REQ-2823](REQ-2823-plain-renderer-carries-same-information.md) | The plain renderer MUST carry the same information as the terminal renderer: every step, every tool call with its policy decision, and the totals. | approved |
| [REQ-2824](REQ-2824-json-one-object-per-line.md) | The JSON renderer MUST emit one JSON object per line, each with a `type` field. | approved |
| [REQ-2825](REQ-2825-stream-begins-with-schema-version.md) | The stream MUST begin with an event carrying the schema version. | approved |
| [REQ-2826](REQ-2826-live-and-export-share-schema.md) | The live stream and `meow session export --format json` MUST share one schema definition and one version number. | approved |
| [REQ-2827](REQ-2827-persisted-kinds-serialise-identically.md) | Every persisted event kind MUST serialise identically in both the live stream and `meow session export --format json`. | approved |
| [REQ-2828](REQ-2828-json-stdout-carries-only-events.md) | With `--format json`, stdout MUST carry only the event stream. | approved |
| [REQ-2829](REQ-2829-json-diagnostics-go-to-stderr.md) | With `--format json`, diagnostics MUST go to stderr. | approved |
| [REQ-2830](REQ-2830-in-flight-kinds-absent-from-export.md) | A kind that exists only while a run is in flight, such as a text delta, MUST NOT appear in an export. | approved |
| [REQ-2831](REQ-2831-export-emits-only-live-kinds.md) | An export MUST NOT emit a kind the live stream cannot. | approved |
| [REQ-2832](REQ-2832-ctx-out-exposes-ten-calls.md) | `ctx.out` MUST expose exactly `write`, `markdown`, `note`, `warn`, `error`, `step`, `table`, `diff`, `finding`, and `json`. | approved |
| [REQ-2833](REQ-2833-ctx-out-exposes-no-layout-calls.md) | `ctx.out` MUST NOT expose any call that positions the cursor, draws a frame, or paginates. | approved |
| [REQ-2834](REQ-2834-ctx-out-calls-produce-typed-events.md) | Each `ctx.out` call MUST produce a typed event that all three renderers handle. | approved |
| [REQ-2835](REQ-2835-ctx-ask-exposes-three-calls.md) | `ctx.ask` MUST expose exactly `text`, `confirm`, and `select`. | approved |
| [REQ-2836](REQ-2836-ctx-ask-fails-without-terminal.md) | Any `ctx.ask` call MUST fail with an error when stdin is not a terminal or `--yes` was given. | approved |
| [REQ-2837](REQ-2837-ctx-ask-never-blocks-without-terminal.md) | Any `ctx.ask` call MUST NOT block when stdin is not a terminal or `--yes` was given. | approved |
| [REQ-2838](REQ-2838-approval-prompt-overwrites-nothing.md) | An approval prompt MUST NOT overwrite anything already in the transcript. | approved |
| [REQ-2839](REQ-2839-approval-prompt-readable-after-answer.md) | An approval prompt MUST remain readable after it is answered. | approved |
| [REQ-2840](REQ-2840-approval-prompt-shows-call-details.md) | The approval prompt MUST show the tool name, the exact arguments, the matching rule, and the agent and step. | approved |
| [REQ-2841](REQ-2841-approval-prompt-offers-four-answers.md) | The approval prompt MUST offer once, always, deny, and stop. | approved |
| [REQ-2842](REQ-2842-always-lasts-for-current-process.md) | Choosing "always" MUST apply for the current process only, per [R-POLICY-023]. | approved |
| [REQ-2843](REQ-2843-user-commands-at-top-level.md) | A user's agents and tools MUST be reachable at the top level of the command line, without a prefix. | approved |
| [REQ-2844](REQ-2844-built-in-commands-grouped.md) | Built-in commands MUST be grouped under `session`, `auth`, `index`, `pkg`, and `policy`, except `init`, `run`, `check`, `models`, `providers`, `doctor`, `trust`, `completions`, and `version`. | approved |
| [REQ-2845](REQ-2845-dry-run-executes-no-tool.md) | `--dry-run` MUST evaluate policy and plan tool calls without executing any, feeding the model a placeholder result for each. | approved |
| [REQ-2846](REQ-2846-dry-run-records-planned-calls.md) | The `--dry-run` transcript MUST record what would have run and how policy would have decided. | approved |
| [REQ-2847](REQ-2847-dry-run-states-divergence.md) | The `--dry-run` transcript MUST state that the run diverges from a real one after the first tool call, because the model's next move depends on a result it never received. | approved |
| [REQ-2848](REQ-2848-yes-denies-every-ask.md) | `--yes` MUST make every `ask` decision resolve to `deny`, per [R-POLICY-020]. | approved |
| [REQ-2849](REQ-2849-yes-never-more-permissive.md) | `--yes` MUST NOT make any decision more permissive. | approved |
| [REQ-2850](REQ-2850-continue-resumes-latest-session.md) | `--continue` MUST resume the most recent session of the command being invoked, in this workspace. | approved |
| [REQ-2851](REQ-2851-continue-fails-without-session.md) | `--continue` MUST fail if the command being invoked has no session in this workspace, rather than starting a fresh run. | approved |
| [REQ-2852](REQ-2852-exit-code-from-stop-reason.md) | The process exit code MUST be derived from the stop reason and the handler's return value: | approved |
| [REQ-2853](REQ-2853-exit-code-never-reused.md) | An exit code MUST NOT be reused for a different stop reason, so that a shell can branch on it. | approved |
| [REQ-2854](REQ-2854-each-stop-reason-one-code.md) | Every stop reason in [R-AGENT-002] MUST map to exactly one exit code. | approved |
| [REQ-2855](REQ-2855-no-color-disables-colour.md) | `NO_COLOR` MUST disable colour unconditionally, whatever the theme declares. | approved |
| [REQ-2856](REQ-2856-palette-quantised-to-colour-depth.md) | Colour depth MUST be detected and the palette quantised to it, rather than colour being dropped. | approved |
| [REQ-2857](REQ-2857-colour-never-sole-carrier.md) | Colour MUST NOT be the only carrier of meaning. | approved |
| [REQ-2858](REQ-2858-status-carries-word-or-sigil.md) | Every severity and status MUST also carry a word or a sigil. | approved |
| [REQ-2859](REQ-2859-ascii-fallback-for-unsupported-glyphs.md) | A terminal that cannot be shown to support the box-drawing and spinner characters MUST get the ASCII fallback. | approved |
| [REQ-3000](REQ-3000-core-depends-on-no-workspace-crate.md) | `meow-core` MUST NOT depend on any other workspace crate. | approved |
| [REQ-3001](REQ-3001-core-performs-no-input-or-output.md) | `meow-core` MUST NOT perform input or output. | approved |
| [REQ-3002](REQ-3002-store-depends-only-on-core.md) | `meow-store` MUST NOT depend on a workspace crate other than `meow-core`. | approved |
| [REQ-3003](REQ-3003-session-depends-on-store-and-core.md) | `meow-session` MUST NOT depend on a workspace crate other than `meow-store` and `meow-core`. | approved |
| [REQ-3004](REQ-3004-llm-depends-only-on-core.md) | `meow-llm` MUST NOT depend on a workspace crate other than `meow-core`. | approved |
| [REQ-3005](REQ-3005-policy-depends-only-on-core.md) | `meow-policy` MUST NOT depend on a workspace crate other than `meow-core`. | approved |
| [REQ-3006](REQ-3006-agent-depends-on-llm-policy-core.md) | `meow-agent` MUST NOT depend on a workspace crate other than `meow-llm`, `meow-policy` and `meow-core`. | approved |
| [REQ-3007](REQ-3007-agent-never-depends-on-star.md) | `meow-agent` MUST NOT depend on `meow-star`, directly or through another crate, so the engine is tested without a script. | approved |
| [REQ-3008](REQ-3008-index-depends-on-store-and-core.md) | `meow-index` MUST NOT depend on a workspace crate other than `meow-store` and `meow-core`. | approved |
| [REQ-3009](REQ-3009-star-depends-on-agent-and-below.md) | `meow-star` MUST NOT depend on a workspace crate other than `meow-agent` and the crates `meow-agent` may depend on. What it needs from the terminal, the session log and the index arrives as a trait, so a test drives a handler with no terminal and no account. | approved |
| [REQ-3010](REQ-3010-ui-depends-only-on-core.md) | `meow-ui` MUST NOT depend on a workspace crate other than `meow-core`, so a renderer runs from a recorded log. | approved |
| [REQ-3011](REQ-3011-cli-may-depend-on-every-crate.md) | `meow-cli` MAY depend on every other workspace crate. | approved |

By topic:

- agent: REQ-1000, REQ-1001, REQ-1002, REQ-1003, REQ-1004, REQ-1005, REQ-1006, REQ-1007, REQ-1008, REQ-1009, REQ-1010, REQ-1011, REQ-1012, REQ-1013, REQ-1014, REQ-1015, REQ-1016, REQ-1017, REQ-1018, REQ-1019, REQ-1020, REQ-1021, REQ-1022, REQ-1023, REQ-1024, REQ-1025, REQ-1026, REQ-1027, REQ-1028, REQ-1029, REQ-1030, REQ-1031, REQ-1032, REQ-1033, REQ-1034, REQ-1035, REQ-1036, REQ-1037, REQ-1038, REQ-1039, REQ-1040, REQ-1041, REQ-1042, REQ-1043, REQ-1044, REQ-1045, REQ-1046, REQ-1047, REQ-1048, REQ-1049, REQ-1050, REQ-1051, REQ-1052, REQ-1053, REQ-1054, REQ-1055, REQ-1056, REQ-1057, REQ-1058, REQ-1059, REQ-1060, REQ-1061, REQ-1062, REQ-1063, REQ-1064, REQ-1065, REQ-1066, REQ-1067, REQ-1068, REQ-1069, REQ-1070, REQ-1071, REQ-1072, REQ-1073, REQ-1074, REQ-1075, REQ-1076, REQ-1077, REQ-1078
- architecture: REQ-3000, REQ-3001, REQ-3002, REQ-3003, REQ-3004, REQ-3005, REQ-3006, REQ-3007, REQ-3008, REQ-3009, REQ-3010, REQ-3011
- auth: REQ-1200, REQ-1201, REQ-1202, REQ-1203, REQ-1204, REQ-1205, REQ-1206, REQ-1207, REQ-1208, REQ-1209, REQ-1210, REQ-1211, REQ-1212, REQ-1213, REQ-1214, REQ-1215, REQ-1216, REQ-1217, REQ-1218, REQ-1219, REQ-1220, REQ-1221, REQ-1222, REQ-1223, REQ-1224, REQ-1225, REQ-1226, REQ-1227, REQ-1228, REQ-1229, REQ-1230, REQ-1231, REQ-1232, REQ-1233
- index: REQ-1400, REQ-1401, REQ-1402, REQ-1403, REQ-1404, REQ-1405, REQ-1406, REQ-1407, REQ-1408, REQ-1409, REQ-1410, REQ-1411, REQ-1412, REQ-1413, REQ-1414, REQ-1415, REQ-1416, REQ-1417, REQ-1418, REQ-1419, REQ-1420, REQ-1421, REQ-1422, REQ-1423, REQ-1424, REQ-1425, REQ-1426, REQ-1427, REQ-1428, REQ-1429, REQ-1430, REQ-1431, REQ-1432, REQ-1433, REQ-1434, REQ-1435, REQ-1436, REQ-1437, REQ-1438, REQ-1439
- llm: REQ-1600, REQ-1601, REQ-1602, REQ-1603, REQ-1604, REQ-1605, REQ-1606, REQ-1607, REQ-1608, REQ-1609, REQ-1610, REQ-1611, REQ-1612, REQ-1613, REQ-1614, REQ-1615, REQ-1616, REQ-1617, REQ-1618, REQ-1619, REQ-1620, REQ-1621, REQ-1622, REQ-1623, REQ-1624, REQ-1625, REQ-1626, REQ-1627, REQ-1628, REQ-1629, REQ-1630, REQ-1631, REQ-1632, REQ-1633, REQ-1634, REQ-1635, REQ-1636, REQ-1637, REQ-1638, REQ-1639, REQ-1640, REQ-1641, REQ-1642, REQ-1643, REQ-1644, REQ-1645, REQ-1646, REQ-1647, REQ-1648, REQ-1649, REQ-1650, REQ-1651, REQ-1652, REQ-1653
- pkg: REQ-1800, REQ-1801, REQ-1802, REQ-1803, REQ-1804, REQ-1805, REQ-1806, REQ-1807, REQ-1808, REQ-1809, REQ-1810, REQ-1811, REQ-1812, REQ-1813, REQ-1814, REQ-1815, REQ-1816, REQ-1817, REQ-1818, REQ-1819, REQ-1820, REQ-1821, REQ-1822, REQ-1823, REQ-1824, REQ-1825, REQ-1826, REQ-1827, REQ-1828, REQ-1829
- policy: REQ-2000, REQ-2001, REQ-2002, REQ-2003, REQ-2004, REQ-2005, REQ-2006, REQ-2007, REQ-2008, REQ-2009, REQ-2010, REQ-2011, REQ-2012, REQ-2013, REQ-2014, REQ-2015, REQ-2016, REQ-2017, REQ-2018, REQ-2019, REQ-2020, REQ-2021, REQ-2022, REQ-2023, REQ-2024, REQ-2025, REQ-2026, REQ-2027, REQ-2028, REQ-2029, REQ-2030, REQ-2031, REQ-2032, REQ-2033, REQ-2034, REQ-2035, REQ-2036, REQ-2037, REQ-2038, REQ-2039, REQ-2040, REQ-2041, REQ-2042, REQ-2043, REQ-2044, REQ-2045, REQ-2046, REQ-2047, REQ-2048, REQ-2049, REQ-2050, REQ-2051, REQ-2052, REQ-2053
- session: REQ-2200, REQ-2201, REQ-2202, REQ-2203, REQ-2204, REQ-2205, REQ-2206, REQ-2207, REQ-2208, REQ-2209, REQ-2210, REQ-2211, REQ-2212, REQ-2213, REQ-2214, REQ-2215, REQ-2216, REQ-2217, REQ-2218, REQ-2219, REQ-2220, REQ-2221, REQ-2222, REQ-2223, REQ-2224, REQ-2225, REQ-2226, REQ-2227, REQ-2228, REQ-2229, REQ-2230, REQ-2231, REQ-2232, REQ-2233, REQ-2234, REQ-2235, REQ-2236, REQ-2237, REQ-2238, REQ-2239, REQ-2240, REQ-2241, REQ-2242, REQ-2243, REQ-2244, REQ-2245, REQ-2246, REQ-2247, REQ-2248, REQ-2249, REQ-2250, REQ-2251, REQ-2252, REQ-2253, REQ-2254, REQ-2255, REQ-2256, REQ-2257, REQ-2258, REQ-2259, REQ-2260, REQ-2261, REQ-2262, REQ-2263, REQ-2264
- star: REQ-2400, REQ-2401, REQ-2402, REQ-2403, REQ-2404, REQ-2405, REQ-2406, REQ-2407, REQ-2408, REQ-2409, REQ-2410, REQ-2411, REQ-2412, REQ-2413, REQ-2414, REQ-2415, REQ-2416, REQ-2417, REQ-2418, REQ-2419, REQ-2420, REQ-2421, REQ-2422, REQ-2423, REQ-2424, REQ-2425, REQ-2426, REQ-2427, REQ-2428, REQ-2429, REQ-2430, REQ-2431, REQ-2432, REQ-2433, REQ-2434, REQ-2435, REQ-2436, REQ-2437, REQ-2438, REQ-2439, REQ-2440, REQ-2441, REQ-2442, REQ-2443, REQ-2444, REQ-2445, REQ-2446, REQ-2447, REQ-2448, REQ-2449, REQ-2450, REQ-2451, REQ-2452, REQ-2453, REQ-2454, REQ-2455, REQ-2456, REQ-2457, REQ-2458, REQ-2459, REQ-2460, REQ-2461, REQ-2462, REQ-2463, REQ-2464, REQ-2465, REQ-2466, REQ-2467, REQ-2468, REQ-2469, REQ-2470, REQ-2471, REQ-2472, REQ-2473, REQ-2474, REQ-2475, REQ-2476, REQ-2477, REQ-2478, REQ-2479, REQ-2480, REQ-2481, REQ-2482, REQ-2483, REQ-2484, REQ-2485, REQ-2486, REQ-2487, REQ-2488, REQ-2489, REQ-2490, REQ-2491, REQ-2492, REQ-2493, REQ-2494, REQ-2495, REQ-2496, REQ-2497, REQ-2498, REQ-2499, REQ-2500, REQ-2501, REQ-2502, REQ-2503, REQ-2504, REQ-2505, REQ-2506, REQ-2507, REQ-2508, REQ-2509, REQ-2510, REQ-2511, REQ-2512, REQ-2513, REQ-2514, REQ-2515, REQ-2516, REQ-2517, REQ-2518, REQ-2519, REQ-2520, REQ-2521, REQ-2522, REQ-2523, REQ-2524, REQ-2525, REQ-2526, REQ-2527, REQ-2528, REQ-2529, REQ-2530, REQ-2531, REQ-2532, REQ-2533, REQ-2534, REQ-2535, REQ-2536, REQ-2537, REQ-2538, REQ-2539, REQ-2540, REQ-2541, REQ-2542, REQ-2543, REQ-2544, REQ-2545, REQ-2546, REQ-2547, REQ-2548, REQ-2549, REQ-2550, REQ-2551, REQ-2552, REQ-2553
- store: REQ-2600, REQ-2601, REQ-2602, REQ-2603, REQ-2604, REQ-2605, REQ-2606, REQ-2607, REQ-2608, REQ-2609, REQ-2610, REQ-2611, REQ-2612, REQ-2613, REQ-2614, REQ-2615, REQ-2616, REQ-2617, REQ-2618, REQ-2619, REQ-2620, REQ-2621, REQ-2622, REQ-2623, REQ-2624, REQ-2625, REQ-2626, REQ-2627, REQ-2628, REQ-2629, REQ-2630, REQ-2631, REQ-2632, REQ-2633, REQ-2634, REQ-2635, REQ-2636, REQ-2637, REQ-2638, REQ-2639, REQ-2640, REQ-2641, REQ-2642, REQ-2643
- tui: REQ-2800, REQ-2801, REQ-2802, REQ-2803, REQ-2804, REQ-2805, REQ-2806, REQ-2807, REQ-2808, REQ-2809, REQ-2810, REQ-2811, REQ-2812, REQ-2813, REQ-2814, REQ-2815, REQ-2816, REQ-2817, REQ-2818, REQ-2819, REQ-2820, REQ-2821, REQ-2822, REQ-2823, REQ-2824, REQ-2825, REQ-2826, REQ-2827, REQ-2828, REQ-2829, REQ-2830, REQ-2831, REQ-2832, REQ-2833, REQ-2834, REQ-2835, REQ-2836, REQ-2837, REQ-2838, REQ-2839, REQ-2840, REQ-2841, REQ-2842, REQ-2843, REQ-2844, REQ-2845, REQ-2846, REQ-2847, REQ-2848, REQ-2849, REQ-2850, REQ-2851, REQ-2852, REQ-2853, REQ-2854, REQ-2855, REQ-2856, REQ-2857, REQ-2858, REQ-2859
<!-- /meow-flow index -->
<!-- markdownlint-enable MD013 -->
