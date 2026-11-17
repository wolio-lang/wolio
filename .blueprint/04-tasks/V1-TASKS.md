`.blueprint/04-tasks/V1-TASKS.md`
# V1 Task Contracts (`M01`–`M07`: `T001`–`T074`)

Normative overview and quick-reference index for [BASELINE-V1.md](../BASELINE-V1.md),
[CORE_V1.md](../02-specifications/language/CORE_V1.md), and
[IMPLEMENTATION_SPECIFICATION_V1.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V1.md).
Every V1 task (`T001`–`T074`) has its own complete, standalone self-contained
contract file linked below:

- **M01 (`T001`–`T004`):** [T001-workspace-skeleton.md](T001-workspace-skeleton.md), [T002-minimal-lexer.md](T002-minimal-lexer.md), [T003-minimal-parser-ast.md](T003-minimal-parser-ast.md), [T004-diagnostics-cli.md](T004-diagnostics-cli.md)
- **M02 (`T020`–`T024`):** [T020-resolver-visibility.md](T020-resolver-visibility.md), [T021-nominal-record-enum-types.md](T021-nominal-record-enum-types.md), [T022-generics-match-result.md](T022-generics-match-result.md), [T023-semantic-identity-graph.md](T023-semantic-identity-graph.md), [T024-interpreter-run.md](T024-interpreter-run.md)
- **M03 (`T030`–`T034`):** [T030-contract-predicates-guards.md](T030-contract-predicates-guards.md), [T031-errors-effects-dependencies.md](T031-errors-effects-dependencies.md), [T032-test-runner-evidence-verify.md](T032-test-runner-evidence-verify.md), [T033-inspect-impact-compatibility.md](T033-inspect-impact-compatibility.md), [T034-trial-examples-pilot.md](T034-trial-examples-pilot.md)
- **M04 (`T040`–`T044`):** [T040-agent-envelopes-replay.md](T040-agent-envelopes-replay.md), [T041-typed-plans-ast-diffs.md](T041-typed-plans-ast-diffs.md), [T042-scope-approval-tiers.md](T042-scope-approval-tiers.md), [T043-transactional-apply-recovery.md](T043-transactional-apply-recovery.md), [T044-repair-finalize-review-analyzer.md](T044-repair-finalize-review-analyzer.md)
- **M05 (`T050`–`T053`):** [T050-typed-ir-verifier.md](T050-typed-ir-verifier.md), [T051-llvm-lowering.md](T051-llvm-lowering.md), [T052-runtime-abi-linking.md](T052-runtime-abi-linking.md), [T053-native-build-parity.md](T053-native-build-parity.md)
- **M06 (`T060`–`T063`):** [T060-benchmark-corpus.md](T060-benchmark-corpus.md), [T061-baseline-runners.md](T061-baseline-runners.md), [T062-paired-benchmark-runs.md](T062-paired-benchmark-runs.md), [T063-benchmark-analysis-report.md](T063-benchmark-analysis-report.md)
- **M07 (`T070`–`T074`):** [T070-git-resolution-lock.md](T070-git-resolution-lock.md), [T071-package-integrity-offline.md](T071-package-integrity-offline.md), [T072-formatter.md](T072-formatter.md), [T073-language-server.md](T073-language-server.md), [T074-release-distribution.md](T074-release-distribution.md)

These tasks become eligible upon owner baseline acceptance recorded in
`.execution/baseline-acceptance.md`.

---

## M01 — Frontend (`T001`–`T004`)

### T001: Establish Rust workspace and crate skeleton
- **Depends on:** owner acceptance of the baseline | **Blocks:** `T002`
- **Bounded areas:** repository root manifests, toolchain/CI configuration, M01 crate skeletons
- **Acceptance families:** common gates (`ACCEPTANCE_MATRIX`)
- **Self-contained contract:**
  - Create the minimal buildable Rust workspace and crate boundaries for `M01` (`crates/wolio-diagnostics`, `crates/wolio-ast`, `crates/wolio-lexer`, `crates/wolio-parser`, `crates/wolio-cli`) without implementing language behavior.
  - Pin `rust-toolchain.toml`, configure root `Cargo.toml`, and set up Linux x86_64 CI workflow running common format, clippy, and test gates.
  - Initialize the `tests/conformance/` directory structure.

### T002: Implement lexer for minimal source slice and V1–V3 keyword reservation
- **Depends on:** `T001` | **Blocks:** `T003`
- **Bounded areas:** lexer crate (`crates/wolio-lexer`), shared diagnostic values
- **Acceptance families:** `FRONT` (lexical cases)
- **Self-contained contract:**
  - Tokenize complete `CORE_V1` lexical vocabulary and reserve all V1 (22), V2, and V3 (24 forward-reserved) keywords with deterministic zero-based UTF-8 byte spans.
  - Parse integers and JSON-escaped strings; reject invalid UTF-8, unescaped control chars, and reserved keywords used as identifiers with `WOLIO-SYNTAX-001`.

### T003: Implement parser and AST for CORE_V1 grammar
- **Depends on:** `T002` | **Blocks:** `T004`
- **Bounded areas:** parser and AST crates (`crates/wolio-parser`, `crates/wolio-ast`)
- **Acceptance families:** `FRONT` (grammar cases)
- **Self-contained contract:**
  - Parse all `CORE_V1` token streams into a deterministic, spanned AST.
  - Enforce explicit precedence and reject chained comparisons (`a == b == c`, `a < b < c`), out-of-range integer literals, and missing parentheses on `if`/`match` conditions with `WOLIO-SYNTAX-001`.

### T004: Add deterministic diagnostics and CLI `init`/`check`
- **Depends on:** `T003` | **Blocks:** `T020` (M02 entry)
- **Bounded areas:** CLI crate (`crates/wolio-cli`), shared diagnostic values, project discovery
- **Acceptance families:** `FRONT` (CLI cases); completes M01
- **Self-contained contract:**
  - Expose project initialization (`wolio init`) and frontend checking (`wolio check`) with canonical JSON envelopes (`--json`).
  - Classify diagnostics according to the 6-tier hierarchy (`DIAGNOSTIC_SPEC.md`) and emit `not_run` semantic evidence records.

---

## M02 — Executable Core (`T020`–`T024`)

### T020: Resolve modules, imports, visibility and cycle checks
- **Depends on:** `T004` | **Blocks:** `T021`
- **Bounded areas:** `crates/wolio-semantic` (resolver submodule), semantic support
- **Acceptance families:** `SEM` (resolution, visibility, cycles)
- **Self-contained contract:**
  - Resolve every name to exactly one declaration or binding; enforce module uniqueness, qualified import rules, and lexical scope.
  - Detect and reject import cycles and recursive operation call cycles with `WOLIO-RESOLVE-001`. Enforce 1-to-1 `operation` to `implementation` pairing in the same module.

### T021: Check nominal, record and enum types
- **Depends on:** `T020` | **Blocks:** `T022`
- **Bounded areas:** `crates/wolio-semantic` (types submodule), semantic support
- **Acceptance families:** `SEM` (constructors, types)
- **Self-contained contract:**
  - Statically type non-generic `CORE_V1` declarations: nominal types (`CustomerId(1)`, `.value`), records, enums (`E::V`, `E::V(x)`), operators, lets, blocks, and branches.
  - Reject recursive value types and non-`Unit` statement expression discards (`WOLIO-TYPE-001` / `WOLIO-LINT-006`).

### T022: Type builtin generics, exhaustive match and Result handling
- **Depends on:** `T021` | **Blocks:** `T023`
- **Bounded areas:** `crates/wolio-semantic` (types submodule)
- **Acceptance families:** `SEM` (builtin generics, matches, Result)
- **Self-contained contract:**
  - Type `Option<T>`, `Result<T, E>`, and `unit()`; reject all other builtin generic applications.
  - Enforce exhaustive pattern matching without wildcards/guards and enforce the Result-handling obligation (must match or forward).

### T023: Compute deterministic identities, snapshot graph and provenance model
- **Depends on:** `T022` | **Blocks:** `T024`
- **Bounded areas:** `crates/wolio-semantic` (graph submodule), CLI (diagnostic targets)
- **Acceptance families:** `ID`
- **Self-contained contract:**
  - Assign deterministic `sym_` SHA-256 declaration and clause IDs over length-prefixed domain fields (`wolio.symbol.v1`).
  - Build the topologically sorted typed semantic graph and snapshot hash; define the change provenance model without external UUID registers.

### T024: Implement the interpreter, runtime and `run` command
- **Depends on:** `T023` | **Blocks:** `T030` (M03 entry)
- **Acceptance families:** `RUN`; completes M02
- **Bounded areas:** runtime/interpreter (`crates/wolio-runtime`), CLI (`crates/wolio-cli`)
- **Self-contained contract:**
  - Execute checked `CORE_V1` programs from the manifest entrypoint with checked 64-bit integer arithmetic (overflow faults with `WOLIO-RUNTIME-001`).
  - Support `console::print` as sole host adapter; reject operations with `requires`/`ensures` at M02 with `WOLIO-FEATURE-001`.

---

## M03 — Trial (`T030`–`T034`)

### T030: Check contract predicates and enforce runtime guards
- **Depends on:** `T024` | **Blocks:** `T031`
- **Bounded areas:** `crates/wolio-semantic` (contracts submodule), runtime (`crates/wolio-runtime`)
- **Acceptance families:** `CONTRACT`
- **Self-contained contract:**
  - Statically validate `requires` and `ensures` predicates (type `Bool`, operation parameter/`result` access).
  - Evaluate predicates in source order as runtime guards around every operation call, raising structured contract violation diagnostics on failure.

### T031: Enforce declared errors, transitive effects and operation dependencies
- **Depends on:** `T030` | **Blocks:** `T032`
- **Bounded areas:** `crates/wolio-semantic` (effects submodule)
- **Acceptance families:** `EFFECT`
- **Self-contained contract:**
  - Statically check conservative `Result::Err` variants against declared `errors [...]`.
  - Statically enforce transitive `effects [...]` upper bounds and explicit `dependencies [...]` declarations for all operation calls (`WOLIO-DEPENDENCY-001`).

### T032: Implement test discovery, runner, evidence model and `verify` CLI
- **Depends on:** `T031` | **Blocks:** `T033`
- **Bounded areas:** verification (`crates/wolio-verify`), CLI (`crates/wolio-cli`)
- **Acceptance families:** `EVID`
- **Self-contained contract:**
  - Discover and execute `test` declarations deterministically with contract guards active.
  - Implement obligation evidence records bound to snapshot/declaration hashes; expose `wolio test` and `wolio verify` (including `--fuzz` scalar boundary mode).

### T033: Implement `inspect`, `impact` and contract compatibility analysis
- **Depends on:** `T032` | **Blocks:** `T034`
- **Bounded areas:** semantic/query (`crates/wolio-semantic`), CLI (`crates/wolio-cli`)
- **Acceptance families:** `QUERY`
- **Self-contained contract:**
  - Expose read-only `wolio inspect` (symbol or full package graph) and `wolio impact --before <p1> --after <p2>`.
  - Classify contract compatibility between snapshots and perform reverse-reachability graph traversal.

### T034: Deliver M03 trial examples, user documentation and pilot benchmark
- **Depends on:** `T033` | **Blocks:** `T040`, `T050`
- **Bounded areas:** examples, user documentation, `benchmarks/pilot`
- **Acceptance families:** `TRIAL`; completes M03
- **Self-contained contract:**
  - Ship three runnable examples (discount, nominal ID lookup, order total), a deliberate contract-failing fixture, and three change cycles.
  - Publish trial user documentation and execute the 6-task pilot benchmark with raw archived run results.

---

## M04 — Agent (`T040`–`T044`)

### T040: Implement local agent JSONL transport, query methods and replay log
- **Depends on:** `T034` | **Blocks:** `T041`
- **Bounded areas:** agent (`crates/wolio-agent`), CLI (`crates/wolio-cli`)
- **Acceptance families:** `AGENT` (versions, query, replay)
- **Self-contained contract:**
  - Implement local stdin/stdout JSON-lines server (`wolio agent`, protocol version 1, 1 MiB line limit).
  - Expose snapshot-checked `inspect` (including Context Spine queries), `impact`, and `verify` query methods; maintain append-only idempotent `.wolio-log/` request/response replay ledger.

### T041: Implement typed mutation plans, canonical AST diffs and identity links
- **Depends on:** `T040` | **Blocks:** `T042`
- **Bounded areas:** agent (`crates/wolio-agent`), semantic
- **Acceptance families:** `AGENT` (plan, raw-text rejection, old/new ID mapping)
- **Self-contained contract:**
  - Validate typed AST mutation plans under `MUTATION_SCHEMA_V1` (`create_declaration`, `replace_implementation`, `replace_contract`, `rename_symbol`, `move_symbol`, `add_test`, `delete_symbol`).
  - Reject raw-text source edits; compute plan hash and canonical AST diff; maintain atomic reference updates and old/new `sym_` provenance mappings.

### T042: Enforce path/symbol scope, capability policy and risk-tier approvals
- **Depends on:** `T041` | **Blocks:** `T043`
- **Bounded areas:** agent (`crates/wolio-agent`)
- **Acceptance families:** `AGENT` (deny, traversal, tier, expiry, revocation)
- **Self-contained contract:**
  - Enforce deny-by-default path/symbol scope and macro-capability envelopes; reject path traversal (`..`) and symlink escapes with `WOLIO-SCOPE-001`.
  - Calculate risk tiers (`AUTO` for all internal evolutions within capability envelopes, `POST_REVIEW` for non-breaking additions, `HUMAN_REQUIRED` strictly for capability escalation, breaking public API changes, or resource budget breaches); validate owner approval records (<= 24h UTC expiry, single-use).

### T043: Implement staged verification, CAS publication and journal crash recovery
- **Depends on:** `T042` | **Blocks:** `T044`
- **Bounded areas:** agent (`crates/wolio-agent`), filesystem transaction/journal adapter
- **Acceptance families:** `AGENT` (crash recovery, concurrent-edit preservation)
- **Self-contained contract:**
  - Stage changes in an isolated temporary workspace and execute pre-publication verification.
  - Publish atomically via exclusive mutation lock, compare-and-swap snapshot verification, and write-ahead journal recovery that never discards concurrent user edits.

### T044: Implement MCTS state rollback repair, Context Spine generator, `finalize`, review disposition and anti-workaround analyzer
- **Depends on:** `T043` | **Blocks:** `T060`, `T070`
- **Bounded areas:** agent (`crates/wolio-agent`), `crates/wolio-semantic` (analyzer submodule), verification (`crates/wolio-verify`)
- **Acceptance families:** `AGENT` (repair, finalize, review, spine, auto-prune); `ANALYZER`; completes M04
- **Self-contained contract:**
  - Enforce V1 analyzer rules (`WOLIO-ANALYZER-001`..`005`, including Zero-Unsafe Invariant, swallowed error rejection, Mechanical Auto-Prune pass for unused dependencies/imports, constant folding, and test-literal mirroring) and 6 hyper-strict lint rules (`WOLIO-LINT-001`..`006`).
  - Implement `finalize` evidence binding, `POST_REVIEW` disposition lifecycle, Context Spine generator (`.wolio/context-spine.json` ≤ 2,000 tokens), and MCTS search-tree exploration with transactional state rollback.

---

## M05 — Native (`T050`–`T053`)

### T050: Implement typed IR lowering and the `VerifiedWolioIr` structural verifier
- **Depends on:** `T034` | **Blocks:** `T051`
- **Bounded areas:** IR (`crates/wolio-ir`)
- **Acceptance families:** `NATIVE` (invalid IR and verified boundary)
- **Self-contained contract:**
  - Lower checked semantic expressions into typed SSA-like IR with explicit basic blocks, calls, variant tags, and arithmetic/contract guards.
  - Implement structural IR verifier that validates types, control-flow edges, and guard invariants, emitting the opaque `VerifiedWolioIr` token.

### T051: Pin LLVM toolchain and lower `VerifiedWolioIr` arithmetic and branches
- **Depends on:** `T050` | **Blocks:** `T052`
- **Bounded areas:** codegen (`crates/wolio-codegen`), toolchain/CI configuration
- **Acceptance families:** `NATIVE` (arithmetic, branches, target check)
- **Self-contained contract:**
  - Pin exact LLVM major version and build digest; accept only `VerifiedWolioIr` into `wolio-codegen`.
  - Emit verified textual LLVM IR (`.ll`) and invoke pinned `clang` to produce Linux x86_64 object code for scalars, control flow, and checked arithmetic (pure Rust codegen, zero `llvm-sys` / `inkwell` C++ build risk).

### T052: Document C/runtime ABI and link records, enums, strings, console and guards
- **Depends on:** `T051` | **Blocks:** `T053`
- **Bounded areas:** runtime (`crates/wolio-runtime`), codegen (`crates/wolio-codegen`)
- **Acceptance families:** `NATIVE` (records, enums, strings, console, guards)
- **Self-contained contract:**
  - Define the V1 native C/runtime ABI for nominal wrappers, records, tagged enums, and reference-counted UTF-8 strings.
  - Implement native runtime adapter for `console::print` and structured abort handlers (`WOLIO-RUNTIME-001`, `WOLIO-CONTRACT-002`..`004`); link runnable Linux x86_64 binaries.

### T053: Expose `wolio build` and pass interpreter/native release parity
- **Depends on:** `T052` | **Blocks:** `T070`
- **Bounded areas:** CLI (`crates/wolio-cli`), `tests/native`
- **Acceptance families:** `NATIVE` (all); completes M05
- **Self-contained contract:**
  - Expose `wolio build` CLI command in human and `--json` modes.
  - Build automated differential test harness (`tests/native`) proving 100% output, fault category, and contract parity between interpreter and release-optimized native builds.

---

## M06 — Evaluation (`T060`–`T063`)

### T060: Freeze 10-task contract & protocol benchmark corpus, prompts and hidden tests
- **Depends on:** `T044` | **Blocks:** `T061`
- **Bounded areas:** `benchmarks/corpus`
- **Acceptance families:** `BENCH` (corpus and 10-task eligibility)
- **Self-contained contract:**
  - Freeze the 10-task contract-and-protocol benchmark corpus, prompts, starter workspaces, and independent hidden acceptance tests across Wolio, Go, Rust, and TypeScript.
  - Record SHA-256 manifests of all inputs; exclude tasks requiring loops or external I/O (scheduled for V2 `T215`).

### T061: Implement isolated Wolio, Go, Rust and TypeScript benchmark runners
- **Depends on:** `T060` | **Blocks:** `T062`
- **Bounded areas:** `benchmarks/runners`
- **Acceptance families:** `BENCH` (equivalent capabilities and budgets)
- **Self-contained contract:**
  - Build reproducible runner harness running Wolio and baseline languages (Go, Rust, TypeScript) under strictly equivalent constraints.
  - Enforce frozen model settings, 3-repair budget, 10-minute timeout, randomized execution order, and hidden-test isolation; capture detailed telemetry.

### T062: Execute 10 paired benchmark runs per arm and archive raw records
- **Depends on:** `T061` | **Blocks:** `T063`
- **Bounded areas:** `benchmarks/results`
- **Acceptance families:** `BENCH` (complete runs including failures)
- **Self-contained contract:**
  - Execute 10 contract-first tasks × 5 paired runs per arm across Wolio, Go, Rust, and TypeScript (200 total runs) in randomized order.
  - Archive all raw JSONL records without selective omission; record every timeout, tool error, and repair exhaustion.

### T063: Compute paired bootstrap intervals and publish the M06 evaluation report
- **Depends on:** `T062` | **Blocks:** none
- **Bounded areas:** `benchmarks/analysis`, evaluation documentation
- **Acceptance families:** `BENCH` (intervals, negative findings); completes M06
- **Self-contained contract:**
  - Compute task success proportions, escaped defects, repair iterations, tokens, time, and cost using 10,000-resample paired bootstrap 95% confidence intervals.
  - Publish the reproducible M06 evaluation report including raw data links, economic tradeoffs, and negative/inconclusive findings.

---

## M07 — Toolchain (`T070`–`T074`)

### T070: Implement Git package resolution, `wolio.lock` and package CLI
- **Depends on:** `T044`, `T053` | **Blocks:** `T071`
- **Bounded areas:** package (`crates/wolio-package`), resolver (`crates/wolio-resolver`), CLI (`crates/wolio-cli`)
- **Acceptance families:** `PKG` (graph, conflicts, imports)
- **Self-contained contract:**
  - Resolve Git repository package dependencies deterministically; generate and parse canonical `wolio.lock` (schema version 1).
  - Enforce flat module imports across locked closure with collision rejection; expose `wolio add`, `remove`, `list`, `info`, and `graph`.

### T071: Enforce Git package integrity, content-addressed cache and offline install
- **Depends on:** `T070` | **Blocks:** `T072`
- **Bounded areas:** package (`crates/wolio-package`), CLI (`crates/wolio-cli`)
- **Acceptance families:** `PKG` (tamper, offline); completes all PKG
- **Self-contained contract:**
  - Enforce HTTPS/SSH Git transports and credential delegation; compute SHA-256 source, contract, and dependency hashes.
  - Implement content-addressed cache under `~/.wolio/cache/`; expose `wolio install` (with `--locked` and `--offline`) and `wolio update`.

### T072: Implement canonical comment-preserving formatter and `wolio fmt`
- **Depends on:** `T071` | **Blocks:** `T073`
- **Bounded areas:** formatter (`crates/wolio-format`), parser, CLI (`crates/wolio-cli`)
- **Acceptance families:** `TOOL` (formatting)
- **Self-contained contract:**
  - Implement canonical, idempotent, comment-preserving source formatter using 4-space indentation and LF line endings.
  - Ensure formatting preserves semantic AST structure, `sym_` IDs, and clause identities; expose `wolio fmt` and read-only `wolio fmt --check`.

### T073: Implement incremental Language Server Protocol (`wolio lsp`) and safe rename
- **Depends on:** `T072` | **Blocks:** `T074`
- **Bounded areas:** LSP (`crates/wolio-lsp`), semantic/query, CLI (`crates/wolio-cli`)
- **Acceptance families:** `TOOL` (editor)
- **Self-contained contract:**
  - Implement stdio `wolio lsp` supporting incremental document sync, diagnostics, hover, definition, and references.
  - Implement workspace rename emitting versioned edits with explicit old/new `sym_` identity mappings, rejecting stale document versions.

### T074: Package Linux x86_64 release artifacts, checksums and clean-install verification
- **Depends on:** `T073` | **Blocks:** none
- **Bounded areas:** release packaging, user/install documentation, CI workflows
- **Acceptance families:** `TOOL` (clean install); completes M07 and full V1
- **Self-contained contract:**
  - Build reproducible Linux x86_64 `.tar.gz` distribution archive with SHA-256 checksums, toolchain/lock metadata, license, and release notes.
  - Enforce release eligibility gate (zero failing checks, zero pending/rejected `POST_REVIEW` records) and verify clean-machine installation.

