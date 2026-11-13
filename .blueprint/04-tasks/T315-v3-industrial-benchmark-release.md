`.blueprint/04-tasks/T315-v3-industrial-benchmark-release.md`
# T315: Execute full W01–W15 industrial benchmark and package Wolio V3 release

| Field | Value |
|---|---|
| Milestone | `M15` |
| Priority | `P0` |
| Depends on | T314 |
| Blocks | none (completes M15 and full V3) |
| Acceptance families | V3-RELEASE, BENCH; completes M15 and full V3 |
| Bounded areas | `benchmarks/v3/`, release packaging, `.execution/reports/T315.md` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T315.md` |

## Objective
Freeze and execute the full comparative industrial benchmark covering all 15 AI
failure and workaround classes (`W01`–`W15` in `AI_BUG_TAXONOMY.md`) across
Wolio V3 (`[language] version = "3"`), Go, Rust, and TypeScript, and package the
verified Wolio V3 Linux x86_64 `.tar.gz` distribution archive, completing M15
and the three-generation Wolio roadmap.

## Authority
- Normative: [BASELINE-V3.md](../BASELINE-V3.md), [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md) (Section 3 & Section 4), [AI_BUG_TAXONOMY.md](../06-project-management/AI_BUG_TAXONOMY.md) (`W01`–`W15`), [DELIVERY_POLICY.md](../03-implementation/DELIVERY_POLICY.md), [RELEASE_CRITERIA.md](../05-quality/RELEASE_CRITERIA.md)
- Implementation layout: [IMPLEMENTATION_SPECIFICATION_V3.md](../03-implementation/IMPLEMENTATION_SPECIFICATION_V3.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V3-RELEASE` and `BENCH`, M15 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Full W01–W15 Industrial Benchmark Corpus (`benchmarks/v3/`):**
  - 30 tasks × 10 paired random seeds × 4 language arms (`1,200` total runs) spanning five 6-task strata covering all 15 `AI_BUG_TAXONOMY.md` failure classes:
    1. **Stratum I (6 tasks — Domain Contracts, Errors, Dependencies & Architecture, `W01`–`W04`, `W06`, `W12`, `W14`, `W15`):** Multi-module domain services under strict contract and layer rules.
    2. **Stratum II (6 tasks — Bounded Iteration, Collections & Exact Financial `Decimal`, `W07`, `W08`):** High-precision ledger, portfolio, and fee settlement pipelines using `std::collections`, `std::math`, and `Decimal`.
    3. **Stratum III (6 tasks — Opaque `Secret<T>`, Crypto & Input Provenance, `W11`):** Webhook HMAC verification (`std::crypto::hmac_sha256`), JSON payload parsing (`std::json::parse`), smart-constructor validation, and secret redaction.
    4. **Stratum IV (6 tasks — Capability I/O, Hermetic `with_mock` Tests & Atomic Transactions, `W09`, `W10`, `W13`):** Stateful multi-account transfers (`resource DbHandle`, `modifies`, `old(...)`, `transaction (db)`) and external HTTP/filesystem workflows tested strictly via `with_mock`.
    5. **Stratum V (6 tasks — Adversarial Anti-Hardcoding & Property Fuzzing, `W05`):** Tasks with sparse unit tests (2 cases) where naive AI agents overfit to test literals and only `property` / `wolio verify --fuzz` (`WOLIO-FUZZ-001`) forces a general algorithm.
- **Statistical Analysis & Reporting (`DELIVERY_PLAN.md`):**
  - Compute 10,000-resample paired bootstrap 95% confidence intervals over tasks for hidden-test pass rate, per-class `W01`–`W15` escaped defect rate, secret-leak/state-corruption security incidence, repair iterations, token count, and wall-clock time.
  - Retain all failed runs, timeouts, and negative/inconclusive findings honestly in `.execution/reports/T315.md`.
- **V3 Linux x86_64 Release Archive:**
  - Package `wolio` binary (supporting `[language] version = "1" | "2" | "3"`, `protocol_version: 1 | 2 | 3`, `std::*`, `verify --fuzz`, native `build`, `fmt`, `lsp`, and offline package manager), `SHA256SUMS`, `TOOLCHAIN.json`, `LICENSE`, and `RELEASE_NOTES_V3.md`.
  - Verify clean-machine installation executing end-to-end sample workflows for `version = "1"`, `version = "2"`, and `version = "3"`.

## Scope
In scope:
- Freezing `benchmarks/v3/corpus/` (30 tasks covering `W01`–`W15`, prompts, starter workspaces, hidden acceptance tests, SHA-256 manifest).
- Executing the paired benchmark across Wolio V3, Go, Rust, and TypeScript and publishing the 10,000-resample bootstrap analysis.
- Packaging and clean-machine verifying the Wolio V3 Linux x86_64 `.tar.gz` release archive.

Out of scope (owner):
- Post-V3 research items (`E03` shared-memory concurrency, `E04` distributed idempotency daemons, `E09` remote multi-tenant server in `POST_V1_PLAN.md`).

## Normative requirements
- All benchmark prompts, starter files, hidden acceptance tests, model/sampling settings, and budgets (3 repair attempts, 10 minutes/task) must be frozen and SHA-256 hashed before any measured run; hidden tests never enter agent context (DELIVERY_PLAN).
- All 15 taxonomy classes (`W01`–`W15`) must be measured and reported with 10,000-resample paired bootstrap 95% confidence intervals, retaining all timeouts, failures, and negative findings without selective omission (AI_BUG_TAXONOMY, DELIVERY_PLAN, ACCEPTANCE_MATRIX).
- V3 release packaging requires 100% of V1, V2, and V3 conformance and `BUG-*` regression suites to pass, zero `pending` or `rejected` `POST_REVIEW` records, verified `SHA256SUMS`, and clean-machine verification of `version = "1"`, `"2"`, and `"3"` projects (DELIVERY_POLICY, RELEASE_CRITERIA).

## Acceptance criteria
Positive and boundary:
- [ ] All 30 W01–W15 benchmark tasks are frozen with SHA-256 manifests and executed across Wolio V3, Go, Rust, and TypeScript (`1,200` paired runs)
- [ ] 10,000-resample paired bootstrap 95% confidence intervals and per-class (`W01`–`W15`) defect rates are published in `.execution/reports/T315.md`
- [ ] Wolio V3 Linux x86_64 `.tar.gz` archive, `SHA256SUMS`, `TOOLCHAIN.json`, `LICENSE`, and `RELEASE_NOTES_V3.md` pass clean-machine installation and execute `version = "1"`, `"2"`, and `"3"` workflows end-to-end

Rejection and failure:
- [ ] Any hidden-test leakage into agent context, selective run omission, unreviewed `POST_REVIEW` record, or checksum mismatch fails release verification

Completion:
- [ ] All M12–M15 gates in `DELIVERY_PLAN` and `ACCEPTANCE_MATRIX` (`V3-REGRESSION`, `V3-STD`, `V3-SECRET`, `V3-IO`, `V3-TX`, `V3-FUZZ`, `V3-RELEASE`, `BENCH`) pass
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T315.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Install the V3 `.tar.gz` archive in a clean Linux x86_64 environment, verify
`SHA256SUMS`, and run the V1, V2, and V3 end-to-end verification workflows.
Evidence: `.execution/reports/T315.md` with benchmark SHA-256 manifest, 10,000-resample bootstrap tables, and V3 release archive digests.

## Dependencies and open items
- Depends on: T314; blocks: none (completes V3).
- Execution prerequisites: model API access and benchmark compute budget (record under `.execution/blockers/` if unavailable; never fabricate or simulate benchmark runs).
- Open: none.

