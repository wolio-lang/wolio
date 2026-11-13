`.blueprint/04-tasks/task-index.md`
# Task index (V1, V2, and V3)

Register of every allocated Wolio task ID across V1 (`M01`–`M07`), V2
(`M08`–`M11`), and V3 (`M12`–`M15`). Read [DOCUMENT_AUTHORITY.md](../DOCUMENT_AUTHORITY.md)
and [BASELINE-V1.md](../BASELINE-V1.md) first, then
[EXECUTION_CONTRACTS.md](EXECUTION_CONTRACTS.md) for shared rules. Live task
states belong only to `.execution/task-status.md`.

---

## Part I — V1 Tasks (`M01`–`M07`, `[language] version = "1"`)

### M01 — Frontend (entry: owner V1 baseline acceptance)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T001 | [T001-workspace-skeleton.md](T001-workspace-skeleton.md) | owner baseline acceptance | common gates |
| T002 | [T002-minimal-lexer.md](T002-minimal-lexer.md) | T001 | FRONT lexical (incl. V1+V2+V3 reserved keywords) |
| T003 | [T003-minimal-parser-ast.md](T003-minimal-parser-ast.md) | T002 | FRONT grammar |
| T004 | [T004-diagnostics-cli.md](T004-diagnostics-cli.md) | T003 | FRONT CLI; all M01 |

### M02 — Executable core (entry: M01)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T020 | [T020-resolver-visibility.md](T020-resolver-visibility.md) | T004 | SEM resolution/cycles (`WOLIO-RESOLVE-001`) |
| T021 | [T021-nominal-record-enum-types.md](T021-nominal-record-enum-types.md) | T020 | SEM constructors/types |
| T022 | [T022-generics-match-result.md](T022-generics-match-result.md) | T021 | SEM matches/Result |
| T023 | [T023-semantic-identity-graph.md](T023-semantic-identity-graph.md) | T022 | ID |
| T024 | [T024-interpreter-run.md](T024-interpreter-run.md) | T023 | RUN; all M02 |

### M03 — Trial (entry: M02)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T030 | [T030-contract-predicates-guards.md](T030-contract-predicates-guards.md) | T024 | CONTRACT |
| T031 | [T031-errors-effects-dependencies.md](T031-errors-effects-dependencies.md) | T030 | EFFECT |
| T032 | [T032-test-runner-evidence-verify.md](T032-test-runner-evidence-verify.md) | T031 | EVID |
| T033 | [T033-inspect-impact-compatibility.md](T033-inspect-impact-compatibility.md) | T032 | QUERY |
| T034 | [T034-trial-examples-pilot.md](T034-trial-examples-pilot.md) | T033 | TRIAL; all M03 |

### M04 — Agent (entry: M03)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T040 | [T040-agent-envelopes-replay.md](T040-agent-envelopes-replay.md) | T034 | AGENT versions/query/replay |
| T041 | [T041-typed-plans-ast-diffs.md](T041-typed-plans-ast-diffs.md) | T040 | AGENT plan/raw-text/ID mapping |
| T042 | [T042-scope-approval-tiers.md](T042-scope-approval-tiers.md) | T041 | AGENT deny/traversal/tier/expiry/revocation |
| T043 | [T043-transactional-apply-recovery.md](T043-transactional-apply-recovery.md) | T042 | AGENT crash/concurrent edits |
| T044 | [T044-repair-finalize-review-analyzer.md](T044-repair-finalize-review-analyzer.md) | T043 | AGENT repair/review; ANALYZER `001`–`005`; all M04 |

### M05 — Native (entry: M03, independent of M04)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T050 | [T050-typed-ir-verifier.md](T050-typed-ir-verifier.md) | T034 | NATIVE invalid IR |
| T051 | [T051-llvm-lowering.md](T051-llvm-lowering.md) | T050 | NATIVE arithmetic/branches |
| T052 | [T052-runtime-abi-linking.md](T052-runtime-abi-linking.md) | T051 | NATIVE records/enums/strings/console/guards |
| T053 | [T053-native-build-parity.md](T053-native-build-parity.md) | T052 | NATIVE all; all M05 |

### M06 — Evaluation (entry: M04)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T060 | [T060-benchmark-corpus.md](T060-benchmark-corpus.md) | T044 | BENCH corpus/eligibility |
| T061 | [T061-baseline-runners.md](T061-baseline-runners.md) | T060 | BENCH capabilities/budgets |
| T062 | [T062-paired-benchmark-runs.md](T062-paired-benchmark-runs.md) | T061 | BENCH complete runs |
| T063 | [T063-benchmark-analysis-report.md](T063-benchmark-analysis-report.md) | T062 | BENCH intervals/negative findings; all M06 |

### M07 — Toolchain (entry: M04 and M05)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T070 | [T070-git-resolution-lock.md](T070-git-resolution-lock.md) | T044, T053 | PKG graph/conflicts/imports |
| T071 | [T071-package-integrity-offline.md](T071-package-integrity-offline.md) | T070 | PKG tamper/offline; all PKG |
| T072 | [T072-formatter.md](T072-formatter.md) | T071 | TOOL formatting |
| T073 | [T073-language-server.md](T073-language-server.md) | T072 | TOOL editor |
| T074 | [T074-release-distribution.md](T074-release-distribution.md) | T073 | TOOL clean install; all M07 |

---

## Part II — V2 Tasks (`M08`–`M11`, `[language] version = "2"`)

### M08 — V1 Bug-Fix & Regression Gate (entry: V1 completion and owner V2 gate acceptance)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T201 | [T201-v1-defect-triage-regressions.md](T201-v1-defect-triage-regressions.md) | T063, T074 | V2-REGRESSION (triage & `BUG-*` fixtures) |
| T202 | [T202-v1-defect-remediation.md](T202-v1-defect-remediation.md) | T201 | V2-REGRESSION, all V1 families |
| T203 | [T203-language-version-isolation-gate.md](T203-language-version-isolation-gate.md) | T202 | V2-REGRESSION; all M08 |

### M09 — Total Bounded Loops, Immutable Collections & Exact Decimal (entry: M08)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T204 | [T204-v2-loop-predicate-parser.md](T204-v2-loop-predicate-parser.md) | T203 | V2-LOOP syntax |
| T205 | [T205-uint-bytes-decimal-scalars.md](T205-uint-bytes-decimal-scalars.md) | T204 | V2-DATA scalars (`UInt`, `Bytes`, `Decimal`) |
| T206 | [T206-bounded-for-loop-guards.md](T206-bounded-for-loop-guards.md) | T205 | V2-LOOP (`WOLIO-CONTRACT-005`/`006`) |
| T207 | [T207-immutable-list-canonical-map.md](T207-immutable-list-canonical-map.md) | T206 | V2-DATA collections (`List<T>`, `Map<K,V>`) |
| T208 | [T208-deterministic-typechecked-format.md](T208-deterministic-typechecked-format.md) | T207 | V2-DATA `format`; all M09 |

### M10 — Pure Predicates, Architecture Policy & Agent Protocol v2 (entry: M09)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T209 | [T209-reusable-pure-predicates.md](T209-reusable-pure-predicates.md) | T208 | V2-PRED |
| T210 | [T210-architecture-module-policy.md](T210-architecture-module-policy.md) | T209 | V2-ARCH (`WOLIO-ARCH-001`) |
| T211 | [T211-agent-protocol-v2-modules.md](T211-agent-protocol-v2-modules.md) | T210 | V2-AGENT (`protocol_version: 2`) |
| T212 | [T212-v2-anti-workaround-analyzer.md](T212-v2-anti-workaround-analyzer.md) | T211 | V2-AGENT, ANALYZER; all M10 |

### M11 — V2 Native Parity, Tooling & Algorithmic Benchmark (entry: M10)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T213 | [T213-v2-ir-llvm-native-lowering.md](T213-v2-ir-llvm-native-lowering.md) | T212 | V2-NATIVE |
| T214 | [T214-v2-formatter-lsp-parity.md](T214-v2-formatter-lsp-parity.md) | T213 | V2-NATIVE, TOOL |
| T215 | [T215-v2-algorithmic-benchmark.md](T215-v2-algorithmic-benchmark.md) | T214 | V2-BENCH, BENCH |
| T216 | [T216-v2-release-distribution.md](T216-v2-release-distribution.md) | T215 | V2-NATIVE, TOOL; all M11 |

---

## Part III — V3 Tasks (`M12`–`M15`, `[language] version = "3"`)

### M12 — V2 Bug-Fix, Memory & Performance Gate (entry: V2 completion and owner V3 gate acceptance)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T301 | [T301-v2-defect-triage-regressions.md](T301-v2-defect-triage-regressions.md) | T216 | V3-REGRESSION (V2 `BUG-*` suite & fixes) |
| T302 | [T302-cow-refcount-collections.md](T302-cow-refcount-collections.md) | T301 | V3-REGRESSION (CoW `List`/`Map` & zero leaks) |
| T303 | [T303-v3-version-isolation-lsp-profiling.md](T303-v3-version-isolation-lsp-profiling.md) | T302 | V3-REGRESSION; all M12 |

### M13 — Industrial Standard Library (`std::*`) & Opaque `Secret<T>` (entry: M12)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T304 | [T304-std-string-math-modules.md](T304-std-string-math-modules.md) | T303 | V3-STD (`std::string`, `std::math`) |
| T305 | [T305-std-collections-json-modules.md](T305-std-collections-json-modules.md) | T304 | V3-STD (`std::collections`, `std::json`) |
| T306 | [T306-opaque-secret-validation-provenance.md](T306-opaque-secret-validation-provenance.md) | T305 | V3-SECRET (`WOLIO-SECURITY-001`) |
| T307 | [T307-std-crypto-module.md](T307-std-crypto-module.md) | T306 | V3-STD, V3-SECRET (`std::crypto`); all M13 |

### M14 — Capability-Gated I/O, Deterministic Test Mocks & Transactions (entry: M13)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T308 | [T308-capability-gated-time-fs-net.md](T308-capability-gated-time-fs-net.md) | T307 | V3-IO (`std::time`, `std::fs`, `std::net`, `[capabilities]`) |
| T309 | [T309-hermetic-test-with-mock-sandbox.md](T309-hermetic-test-with-mock-sandbox.md) | T308 | V3-IO (`with_mock`, `WOLIO-EFFECT-002`) |
| T310 | [T310-linear-resource-modifies-old-snapshots.md](T310-linear-resource-modifies-old-snapshots.md) | T309 | V3-TX (`resource`, `modifies`, `old(...)`, `WOLIO-RESOURCE-001`) |
| T311 | [T311-atomic-transaction-rollback.md](T311-atomic-transaction-rollback.md) | T310 | V3-TX (`transaction` rollback); all M14 |

### M15 — Contract Property Fuzzing, Agent Protocol v3 & Industrial Release (entry: M14)

| ID | Task file | Depends on | Acceptance |
|---|---|---|---|
| T312 | [T312-property-fuzz-shrinking-verifier.md](T312-property-fuzz-shrinking-verifier.md) | T311 | V3-FUZZ (`property`, `wolio verify --fuzz`, `WOLIO-FUZZ-001`) |
| T313 | [T313-agent-protocol-v3-mutations.md](T313-agent-protocol-v3-mutations.md) | T312 | V3-FUZZ, AGENT (`protocol_version: 3`) |
| T314 | [T314-v3-ir-llvm-formatter-lsp-parity.md](T314-v3-ir-llvm-formatter-lsp-parity.md) | T313 | V3-RELEASE, NATIVE, TOOL |
| T315 | [T315-v3-industrial-benchmark-release.md](T315-v3-industrial-benchmark-release.md) | T314 | V3-RELEASE, BENCH (W01–W15); all M15 |
