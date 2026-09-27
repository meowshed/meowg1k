# Decisions

One decision per file. Each was recovered from `docs/spec/`, `docs/design/`,
`docs/philosophy.md` or the forge history on 2026-09-27 and is a draft until the
owner approves it.

<!-- meow-method index -->

182 decisions in all: 182 approved.

| Identifier | What it concluded | Status |
| --- | --- | --- |
| [ADR-0100](ADR-0100-handlers-block-on-the-engine.md) | A handler blocks its own script thread on the async engine | approved |
| [ADR-0101](ADR-0101-handlers-stay-imperative.md) | Handlers stay imperative Starlark, and only concurrency is taken away from them | approved |
| [ADR-0102](ADR-0102-parallel-takes-invocations.md) | Concurrency runs over agent invocations and never over Starlark closures | approved |
| [ADR-0103](ADR-0103-starlark-renders-its-own-errors.md) | Starlark errors keep the diagnostics `starlark-rust` renders | approved |
| [ADR-0104](ADR-0104-configuration-is-not-layered.md) | A workspace's configuration is read alone and never merged with a global one | approved |
| [ADR-0105](ADR-0105-nothing-chooses-the-index-model.md) | Nothing chooses an embedding model for the index on the workspace's behalf | approved |
| [ADR-0106](ADR-0106-yes-resolves-ask-to-deny.md) | Under `--yes` or with no terminal, an `ask` decision resolves to deny | approved |
| [ADR-0107](ADR-0107-always-grant-is-never-persisted.md) | An "always" approval is never written back to `meow.star` | approved |
| [ADR-0108](ADR-0108-approval-shows-exact-arguments.md) | An approval prompt shows the exact call verbatim and names the rule that asked | approved |
| [ADR-0109](ADR-0109-no-chat-interface.md) | meowg1k has no chat interface, and iteration is re-invocation | approved |
| [ADR-0110](ADR-0110-transcript-first-inline-viewport.md) | The terminal renderer commits the transcript to scrollback and redraws only a small live region | approved |
| [ADR-0111](ADR-0111-fresh-session-by-default.md) | A run starts a fresh session unless it names one | approved |
| [ADR-0112](ADR-0112-gc-deletes-whole-sessions.md) | Garbage collection deletes whole sessions and never part of one | approved |
| [ADR-0113](ADR-0113-denials-are-recorded.md) | Every tool call records its policy decision before it runs, denials included | approved |
| [ADR-0114](ADR-0114-store-is-apart-from-sessions.md) | The workspace store and session state are separate scopes, and neither is session metadata | approved |
| [ADR-0115](ADR-0115-two-rebuild-functions.md) | The two rebuild modes are two functions over one iterator | approved |
| [ADR-0116](ADR-0116-live-stream-and-export-share-a-type.md) | The live JSON stream and the session export share one event type | approved |
| [ADR-0117](ADR-0117-meowignore-negations-walked-separately.md) | `.meowignore` negations are walked separately from the ignore set | approved |
| [ADR-0118](ADR-0118-one-openai-shaped-implementation.md) | Every OpenAI-shaped vendor is served by one provider implementation | approved |
| [ADR-0119](ADR-0119-structured-output-is-validated.md) | Structured output reaches the script parsed and validated | approved |
| [ADR-0120](ADR-0120-engine-enforces-nested-budgets.md) | The engine enforces nested budgets, so a fan-out can't exceed the top-level cap | approved |
| [ADR-0121](ADR-0121-ask-fails-without-a-terminal.md) | `ctx.ask` fails instead of blocking when nobody can answer | approved |
| [ADR-0400](ADR-0400-denied-is-its-own-stop-reason.md) | A run a policy stopped ends with `denied`, apart from `tool_aborted` | approved |
| [ADR-0401](ADR-0401-policy-evaluation-gets-resolved-paths.md) | Policy evaluation receives resolved paths and touches nothing | approved |
| [ADR-0402](ADR-0402-a-session-holds-several-runs.md) | A session holds one or more runs, each opened and closed | approved |
| [ADR-0403](ADR-0403-tool-calls-run-in-sequence.md) | Tool calls in one response run in the order the model returned them | approved |
| [ADR-0404](ADR-0404-budget-is-reserved-before-a-call.md) | Budget is reserved before a call, not checked and then charged | approved |
| [ADR-0405](ADR-0405-each-event-commits-on-its-own.md) | Each event commits on its own, and no transaction spans a turn | approved |
| [ADR-0406](ADR-0406-every-payload-goes-to-the-blob-table.md) | The store keeps every payload in the `blobs` table and never inline | approved |
| [ADR-0407](ADR-0407-caching-a-generation-needs-an-opt-in.md) | Caching a generation needs the caller to name it and opt in | approved |
| [ADR-0408](ADR-0408-a-total-with-an-unpriced-call-has-no-cost.md) | A total that includes an unpriced call reports no cost | approved |
| [ADR-0409](ADR-0409-providers-are-tested-against-recordings.md) | Providers are tested against recorded exchanges behind a `Transport` trait | approved |
| [ADR-0410](ADR-0410-a-spent-quota-is-told-by-the-provider-signal.md) | A spent quota is told from a rate limit by the provider's own signal | approved |
| [ADR-0411](ADR-0411-a-call-no-rule-matches-is-denied.md) | A call no rule matches is denied | approved |
| [ADR-0412](ADR-0412-explain-answers-about-rules.md) | `meow policy explain` answers about rules, not outcomes | approved |
| [ADR-0413](ADR-0413-tokens-are-estimated-at-four-characters.md) | Token counts are estimated at four characters a token | approved |
| [ADR-0414](ADR-0414-fan-out-branches-share-the-caller-ledger.md) | A fan-out's branches share the caller's ledger, and the rest stop | approved |
| [ADR-0415](ADR-0415-references-resolve-after-every-file.md) | References resolve after every declaration file has been evaluated | approved |
| [ADR-0416](ADR-0416-arg-patterns-are-wildcards-not-regex.md) | `meow.arg.string(pattern = ...)` accepts `*` and literals, not regular expressions | approved |
| [ADR-0417](ADR-0417-a-markdown-prompt-loads-as-a-module.md) | A `.md` prompt file loads as a module with one symbol | approved |
| [ADR-0418](ADR-0418-an-omitted-budget-axis-keeps-its-default.md) | An omitted budget axis in a declaration keeps the engine's default | approved |
| [ADR-0419](ADR-0419-agent-and-tool-declarations-return-values.md) | `meow.agent` and `meow.tool` return values rather than register names | approved |
| [ADR-0420](ADR-0420-builtins-check-the-phase-not-the-load.md) | Every module loads in both phases, and each builtin refuses a call made during declaration | approved |
| [ADR-0421](ADR-0421-a-handler-is-found-by-name.md) | A tool handler is found by file and symbol, not held as a value | approved |
| [ADR-0422](ADR-0422-terminal-and-session-arrive-as-ports.md) | The terminal, the session log and search arrive in `meow-star` as ports | approved |
| [ADR-0423](ADR-0423-nesting-depth-is-fixed-at-three.md) | The sub-agent nesting depth is 3 and isn't declarable | approved |
| [ADR-0424](ADR-0424-ctx-out-port-takes-one-typed-event.md) | The port behind `ctx.out` takes one typed event | approved |
| [ADR-0425](ADR-0425-the-plain-renderer-writes-text-once.md) | The plain renderer writes a step's text once and drops deltas | approved |
| [ADR-0426](ADR-0426-relay-lives-in-meow-star.md) | The mapping from engine events to view events lives in `meow-star` | approved |
| [ADR-0427](ADR-0427-retry-after-is-read-in-seconds.md) | `Retry-After` is read only in its seconds form | approved |
| [ADR-0428](ADR-0428-workspace-commands-are-top-level.md) | A workspace's agents and tools are top-level commands | approved |
| [ADR-0429](ADR-0429-a-provider-without-a-credential-is-skipped.md) | A provider with no credential is skipped at startup and fails when used | approved |
| [ADR-0430](ADR-0430-an-approval-is-answered-by-a-line.md) | An approval prompt is answered by a line, not a keypress | approved |
| [ADR-0431](ADR-0431-dry-run-still-calls-the-model.md) | `--dry-run` still calls the model and hands it a placeholder result | approved |
| [ADR-0432](ADR-0432-a-fork-is-a-sibling.md) | A fork is a sibling of its origin, not a descendant | approved |
| [ADR-0433](ADR-0433-retention-applies-the-strictest-limit.md) | Retention applies the strictest of its limits | approved |
| [ADR-0434](ADR-0434-size-retention-re-measures-after-each-deletion.md) | The size limit deletes and re-measures one session at a time | approved |
| [ADR-0435](ADR-0435-export-takes-redaction-from-the-caller.md) | Export takes its redaction from the caller | approved |
| [ADR-0436](ADR-0436-continue-resumes-the-commands-own-session.md) | `--continue` resumes the invoked command's own session and never starts a fresh one | approved |
| [ADR-0437](ADR-0437-a-resumed-run-takes-the-current-prompt.md) | A resumed run takes its system prompt from the specification, not the history | approved |
| [ADR-0438](ADR-0438-identifier-entropy-from-clock-and-pid.md) | Session identifier entropy comes from the clock and the process id | approved |
| [ADR-0439](ADR-0439-session-set-writes-a-note.md) | `ctx.session.set` writes a `Note`, not a new event kind | approved |
| [ADR-0440](ADR-0440-chunk-overlap-is-counted-in-lines.md) | Chunk overlap is counted in lines | approved |
| [ADR-0441](ADR-0441-the-walk-uses-the-ignore-crate.md) | The workspace walk uses the `ignore` crate | approved |
| [ADR-0442](ADR-0442-staleness-hash-covers-chunking.md) | The staleness hash covers the chunking parameters as well as the content | approved |
| [ADR-0443](ADR-0443-gemini-drops-schema-keywords-it-rejects.md) | Schema keywords Gemini rejects are dropped before sending | approved |
| [ADR-0444](ADR-0444-structured-output-has-a-none-state.md) | `Structured` has a third state for a provider that produces no text | approved |
| [ADR-0445](ADR-0445-a-command-is-a-list-of-words.md) | A command is a list of words, never one string | approved |
| [ADR-0446](ADR-0446-handlers-reach-markdown-agents-by-name.md) | A handler reaches a markdown agent through `meow.agent_named` | approved |
| [ADR-0447](ADR-0447-toml-encode-refuses-a-null.md) | `toml.encode` refuses a null rather than dropping the key | approved |
| [ADR-0448](ADR-0448-encoders-convert-to-the-target-type.md) | `yaml.encode` and `toml.encode` convert to the target's own value type | approved |
| [ADR-0449](ADR-0449-csv-encode-reads-starlark-values.md) | `csv.encode` reads the Starlark values directly | approved |
| [ADR-0450](ADR-0450-text-search-goes-through-the-search-port.md) | `search.text` and `search.files` go through the search port | approved |
| [ADR-0451](ADR-0451-index-calls-report-counts.md) | `index` calls report counts rather than sentences | approved |
| [ADR-0452](ADR-0452-search-code-is-query-with-a-zero-floor.md) | `search.code` is `index.query` with a floor of zero | approved |
| [ADR-0453](ADR-0453-store-refuses-when-the-database-wont-open.md) | `store` refuses every call when the database won't open | approved |
| [ADR-0454](ADR-0454-an-http-body-over-the-cap-is-cut.md) | An `http` body over the read cap is cut, not refused | approved |
| [ADR-0455](ADR-0455-a-readable-store-is-refused-not-repaired.md) | A credential store readable by others is refused, not repaired | approved |
| [ADR-0456](ADR-0456-store-writes-go-through-a-temporary-beside-it.md) | A store write goes to a temporary file beside the target, then a rename | approved |
| [ADR-0457](ADR-0457-the-key-prompt-echoes-and-says-so.md) | The key prompt echoes what is typed and says so | approved |
| [ADR-0458](ADR-0458-trust-covers-declarations-not-handler-bodies.md) | The trust fingerprint covers declarations, not handler bodies | approved |
| [ADR-0459](ADR-0459-describing-commands-run-without-trust.md) | The describing commands and `meow pkg` run without trust | approved |
| [ADR-0460](ADR-0460-agreeing-to-trust-does-not-run.md) | Agreeing to trust a workspace doesn't then run the command | approved |
| [ADR-0461](ADR-0461-oauth-providers-are-chosen-by-name.md) | Which provider authenticates by OAuth is decided by its name | approved |
| [ADR-0462](ADR-0462-a-refused-renewal-is-fatal.md) | A refused credential renewal is fatal and isn't retried | approved |
| [ADR-0463](ADR-0463-package-hash-is-recomputed-on-load.md) | A package's tree hash is recomputed on every load | approved |
| [ADR-0464](ADR-0464-meow-pkg-stands-in-for-absent-packages.md) | `meow pkg` stands in for a declared package that isn't there yet | approved |
| [ADR-0465](ADR-0465-the-trust-question-lists-packages.md) | The trust question lists the packages a workspace declares | approved |
| [ADR-0466](ADR-0466-cancellation-is-checked-before-a-request.md) | Cancellation is checked before a request as well as raced against it | approved |
| [ADR-0467](ADR-0467-credentials-live-in-the-os-secret-store.md) | Credentials live in the operating system's secret store, with the file as the fallback | approved |
| [ADR-0468](ADR-0468-request-ceilings-are-counted-in-the-database.md) | Request ceilings are counted in the workspace database, shared by every invocation | approved |
| [ADR-0469](ADR-0469-commands-are-starlark-not-compiled-workflows.md) | User commands are Starlark files, not compiled workflows | approved |
| [ADR-0600](ADR-0600-crates-depend-one-way.md) | The workspace's crates depend one way, as CLAUDE.md's table sets out | approved |
| [ADR-0601](ADR-0601-price-lives-in-the-model-declaration.md) | A model's price lives in its declaration | approved |
| [ADR-0602](ADR-0602-a-run-is-observed-through-typed-callbacks.md) | A handler observes a run through typed callbacks, one per kind | approved |
| [ADR-0603](ADR-0603-presets-fold-into-the-model.md) | Presets fold into `meow.model`, which is the named configuration | approved |
| [ADR-0604](ADR-0604-env-require-fails-at-load.md) | `env.require` fails at load, naming the variable | approved |
| [ADR-0605](ADR-0605-crypto-template-and-math-are-left-out.md) | `crypto`, `template` and `math` are left out of the module table | approved |
| [ADR-0606](ADR-0606-no-migration-from-v02-sessions.md) | v0.2.x sessions are not migrated | approved |
| [ADR-0607](ADR-0607-https-trusts-the-native-root-store.md) | HTTPS trusts the operating system's root certificates | approved |
| [ADR-0608](ADR-0608-a-refused-stream-fails-at-the-call.md) | A streaming request the vendor refuses fails at the call | approved |
| [ADR-0609](ADR-0609-connect-timeout-and-no-response-deadline.md) | A provider call times out only while it connects | approved |
| [ADR-0610](ADR-0610-an-explicit-none-is-absent.md) | An explicit `None` for an optional declaration argument means absent | approved |
| [ADR-0611](ADR-0611-policy-reads-arguments-by-name.md) | The policy learns what a tool call touches from its argument names | approved |
| [ADR-0612](ADR-0612-a-tool-result-is-logged-without-its-output.md) | A tool result is logged without the tool's output | approved |
| [ADR-0613](ADR-0613-fs-remove-refuses-the-root-and-the-store.md) | `fs.remove` refuses the workspace root and the store | approved |
| [ADR-0614](ADR-0614-fs-follows-links-inside-the-workspace.md) | `fs` checks a relative path lexically and follows links inside it | approved |
| [ADR-0615](ADR-0615-git-runs-the-users-own-git.md) | `@std//git` runs the user's own `git` as a subprocess | approved |
| [ADR-0616](ADR-0616-git-refuses-values-that-look-like-options.md) | `@std//git` refuses a caller's value that begins with a dash | approved |
| [ADR-0617](ADR-0617-git-commit-takes-only-what-is-staged.md) | `git.commit` takes only what is staged | approved |
| [ADR-0618](ADR-0618-git-status-returns-porcelain-text.md) | `git.status` returns `git`'s porcelain format as text | approved |
| [ADR-0619](ADR-0619-http-refuses-other-schemes.md) | `http` refuses a URL whose scheme isn't `http` or `https` | approved |
| [ADR-0620](ADR-0620-an-escaping-archive-is-caught-twice.md) | A package archive that escapes is caught by two checks | approved |
| [ADR-0621](ADR-0621-a-package-source-is-a-tarball.md) | A package source is a gzipped tarball over HTTP | approved |
| [ADR-0622](ADR-0622-a-fetch-caps-size-and-time.md) | A fetch refuses an archive over 64 MiB and gives up after 120 seconds | approved |
| [ADR-0623](ADR-0623-fs-glob-walks-the-tree-each-call.md) | `fs.glob` walks the whole workspace on each call and skips `.data` | approved |
| [ADR-1000](ADR-1000-sub-agents-are-isolated.md) | A sub-agent sees only the task and the arguments its caller passed | approved |
| [ADR-1001](ADR-1001-compaction-summarises.md) | Compaction summarises the superseded range with a model call | approved |
| [ADR-1002](ADR-1002-compaction-names-its-own-model.md) | The compaction policy names its own model and falls back to the agent's | approved |
| [ADR-1003](ADR-1003-default-budget-values.md) | The default budget is 200,000 tokens, 40 steps and 30 minutes, with cost unbounded | approved |
| [ADR-1004](ADR-1004-runs-return-an-outcome.md) | Every run returns an outcome carrying its stop reason and transcript | approved |
| [ADR-1005](ADR-1005-omitted-arguments-are-not-zero-filled.md) | An omitted argument is never filled with a zero value | approved |
| [ADR-1006](ADR-1006-compaction-runs-in-the-engine.md) | Compaction runs in the engine, not in a userland library | approved |
| [ADR-1007](ADR-1007-sink-errors-handled-identically.md) | A sink error is handled the same way for every event kind | approved |
| [ADR-1200](ADR-1200-credential-resolution-order.md) | A credential resolves from three places in one fixed order | approved |
| [ADR-1201](ADR-1201-listing-never-shows-a-credential.md) | No command shows a stored credential, in full or in part | approved |
| [ADR-1202](ADR-1202-trust-re-asked-on-change.md) | Trust is per workspace and asked again when its declarations change | approved |
| [ADR-1400](ADR-1400-line-oriented-chunking.md) | Chunking is line-oriented | approved |
| [ADR-1401](ADR-1401-hnsw-rs-vector-index.md) | The vector index is `hnsw_rs` | approved |
| [ADR-1402](ADR-1402-prose-indexed-with-path-filter.md) | Prose is indexed alongside code, and a query narrows with a path filter | approved |
| [ADR-1403](ADR-1403-index-in-one-database.md) | The index lives in the one database | approved |
| [ADR-1404](ADR-1404-oversized-chunk-error-names-file.md) | An oversized chunk's error names the file and the line range | approved |
| [ADR-1405](ADR-1405-index-records-embedding-model.md) | The index records the embedding model that built it | approved |
| [ADR-1600](ADR-1600-credential-asked-per-request.md) | A provider asks for its credential on each request | approved |
| [ADR-1601](ADR-1601-cache-control-is-an-optional-hint.md) | Cache control is an optional hint on a message | approved |
| [ADR-1602](ADR-1602-thinking-is-streamed-and-stored.md) | Thinking content is streamed and stored | approved |
| [ADR-1603](ADR-1603-classify-errors-before-retrying.md) | Provider errors are classified, and only transient ones are retried | approved |
| [ADR-1604](ADR-1604-deduplicate-tool-calls-at-provider.md) | Tool calls are deduplicated at the provider boundary | approved |
| [ADR-1605](ADR-1605-no-synthesised-stream-events.md) | A provider without native streaming declares streaming unsupported | approved |
| [ADR-1606](ADR-1606-report-cached-prompt-tokens.md) | A response reports cached prompt tokens as a separate count | approved |
| [ADR-1800](ADR-1800-load-never-fetches.md) | A load never fetches | approved |
| [ADR-1801](ADR-1801-cache-keyed-by-hash.md) | The package cache is keyed by hash and not by name and version | approved |
| [ADR-1802](ADR-1802-package-cannot-see-workspace.md) | A package cannot see the workspace that loads it | approved |
| [ADR-2000](ADR-2000-partial-read-whole-write.md) | A multi-path read is partial and says so, and a multi-path write is all or nothing | approved |
| [ADR-2001](ADR-2001-hosts-selector-for-network-tools.md) | Network tools get a hosts selector now | approved |
| [ADR-2002](ADR-2002-policy-judges-model-tool-calls.md) | Policy governs what a model decided, not what a script did | approved |
| [ADR-2003](ADR-2003-approval-prompt-waits.md) | An approval prompt waits indefinitely by default | approved |
| [ADR-2004](ADR-2004-tool-calls-pass-policy-layer.md) | Every tool call a model makes passes a policy layer in the runtime | approved |
| [ADR-2200](ADR-2200-compaction-supersedes-without-deleting.md) | Compaction supersedes a range of events without deleting it | approved |
| [ADR-2201](ADR-2201-usage-is-typed-fields.md) | A Usage event carries tokens and cost as typed fields | approved |
| [ADR-2202](ADR-2202-short-id-is-random-tail.md) | A session's short identifier is the last eight characters of a time-sortable identifier | approved |
| [ADR-2203](ADR-2203-log-defines-session-state.md) | The event log defines a session's state | approved |
| [ADR-2204](ADR-2204-sessions-can-be-forked.md) | A session can be forked at a sequence number | approved |
| [ADR-2205](ADR-2205-sessions-are-workspace-local.md) | Sessions are workspace-local | approved |
| [ADR-2206](ADR-2206-liveness-is-a-heartbeat.md) | A session's liveness is a heartbeat on its row | approved |
| [ADR-2400](ADR-2400-declaration-reads-environment-only.md) | Declaration files may read the environment and nothing else | approved |
| [ADR-2401](ADR-2401-handlers-never-declare-tools.md) | A handler may not declare a tool | approved |
| [ADR-2402](ADR-2402-re-time-return-one-scale.md) | `re` and `time` return one scale each | approved |
| [ADR-2403](ADR-2403-xml-parses-to-tree.md) | XML is a tree and not a dictionary | approved |
| [ADR-2404](ADR-2404-csv-header-yields-dictionaries.md) | A CSV with a header is a list of dictionaries | approved |
| [ADR-2405](ADR-2405-store-holds-values-not-text.md) | The store holds values and not text | approved |
| [ADR-2406](ADR-2406-missing-index-differs-from-no-results.md) | No index and no results are different answers | approved |
| [ADR-2407](ADR-2407-text-search-needs-no-index.md) | `search.text` does not need an index | approved |
| [ADR-2408](ADR-2408-http-status-is-answer.md) | An HTTP status is an answer and not a failure | approved |
| [ADR-2409](ADR-2409-http-outside-policy-layer.md) | `http` is not behind the policy layer | approved |
| [ADR-2410](ADR-2410-paths-use-forward-slash.md) | Paths are written with `/` on every platform | approved |
| [ADR-2411](ADR-2411-markdown-agents-compose-by-inclusion.md) | Markdown agents compose by inclusion only | approved |
| [ADR-2412](ADR-2412-one-module-table-six-member-context.md) | One module table and a six-member context replace the hand-built contexts | approved |
| [ADR-2413](ADR-2413-arguments-drive-flag-help-schema.md) | `meow.arg` with real types replaces `meow.param` | approved |
| [ADR-2414](ADR-2414-agents-declarable-as-markdown.md) | An agent can be declared as a markdown file | approved |
| [ADR-2600](ADR-2600-plaintext-database-owner-only.md) | The database is plaintext, with owner-only permissions | approved |
| [ADR-2601](ADR-2601-synchronous-is-normal.md) | `synchronous` is `NORMAL` | approved |
| [ADR-2602](ADR-2602-one-database-per-workspace.md) | One SQLite database holds all of a workspace's data | approved |
| [ADR-2603](ADR-2603-failed-writes-propagate.md) | A failed write propagates to the caller | approved |
| [ADR-2604](ADR-2604-content-addressed-payloads.md) | Event payloads are content-addressed | approved |
| [ADR-2800](ADR-2800-live-region-fixed-three-rows.md) | The live region is a fixed three rows, and an approval prompt goes into the transcript | approved |
| [ADR-2801](ADR-2801-json-format-streams-jsonl.md) | `--format json` streams one JSON object per line | approved |
| [ADR-2802](ADR-2802-one-event-stream-three-renderers.md) | One event stream feeds three renderers | approved |
| [ADR-2803](ADR-2803-ctx-out-semantic-calls.md) | `ctx.out` offers ten semantic calls and no layout builtins | approved |
| [ADR-2804](ADR-2804-json-mode-machine-readable.md) | `--format json` gives every run a machine-readable mode | approved |
| [ADR-2805](ADR-2805-exit-code-per-stop-reason.md) | Each stop reason has its own exit code | approved |
| [ADR-2806](ADR-2806-diagnostics-into-scrollback.md) | Diagnostics go into scrollback and never through the live frame | approved |
<!-- /meow-method index -->
