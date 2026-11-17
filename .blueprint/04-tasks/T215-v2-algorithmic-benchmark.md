`.blueprint/04-tasks/T215-v2-algorithmic-benchmark.md`
# T215: Execute 24-task V2 algorithmic, collection and architecture benchmark (`W04`, `W07`, `W08`)

| Field | Value |
|---|---|
| Milestone | `M11` |
| Priority | `P0` |
| Depends on | T214 |
| Blocks | T216 |
| Acceptance families | V2-BENCH, BENCH |
| Bounded areas | `benchmarks/v2/`, `.execution/reports/T215.md` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T215.md` |

## Objective
Freeze and execute the 24-task × 10-seed paired comparative V2 benchmark across
Wolio V2 (`[language] version = "2"`), Go, Rust, and TypeScript, measuring AI
defect prevention on bounded iteration (`W07`), exact `Decimal` financial rules
and reusable predicates (`W08`), and multi-module `[architecture]` boundaries
(`W04`).

## Authority
- Normative: [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md) (Section 2 & Section 4: Benchmark protocol), [BASELINE-V2.md](../BASELINE-V2.md), [AI_BUG_TAXONOMY.md](../06-project-management/AI_BUG_TAXONOMY.md) (`W01`–`W08`, `W12`, `W14`, `W15`)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `V2-BENCH` and `BENCH`, [RELEASE_CRITERIA.md](../05-quality/RELEASE_CRITERIA.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)

## Self-contained implementation context
- **Corpus structure (`benchmarks/v2/`):**
  - 24 tasks divided across four 6-task strata:
    1. **Stratum A (6 tasks — Bounded Iteration & Accumulators, `W07`):** Aggregation, filtering, windowing, and range folding where off-by-one or infinite loop bugs commonly occur.
    2. **Stratum B (6 tasks — Immutable Collections `List<T>` & `Map<K, V>` with Turbofish, `W06`/`W07`):** Canonical-order key aggregation, deduplication, lookup joins, and inventory reconciliation.
    3. **Stratum C (6 tasks — Exact Financial `Decimal` with Explicit `RoundingMode` & Reusable `predicate`s, `W08`):** Multi-tier tax, fee splitting, invoice reconciliation, and basis-point rounding where IEEE-754 float drift or silent truncation fails hidden tests.
    4. **Stratum D (6 tasks — Multi-Module Architecture & Agent Protocol v2, `W04`/`W12`/`W15`):** Adding domain/application modules and imports under `[architecture]` `allow_imports`/`deny_imports` rules via `protocol_version: 2`.
- **Execution & statistical protocol (`DELIVERY_PLAN.md`):**
  - Freeze all 24 task prompts, starter workspaces, and hidden independent acceptance tests (with SHA-256 manifest) before running any measured trial.
  - Run 24 tasks × 10 paired random seeds × 4 language arms (`960` total runs), budget max 3 repair attempts and 10 minutes per task.
  - Compute 10,000-resample paired bootstrap 95% confidence intervals over tasks for hidden-test pass rate, escaped `W01`–`W08` defect rate, repair iterations, token usage, and wall-clock time.

## Scope
In scope:
- Freezing `benchmarks/v2/corpus/` (prompts, starter workspaces, hidden tests, SHA-256 hash manifest).
- Executing the 960 paired runs across Wolio V2, Go, Rust, and TypeScript.
- Computing the 10,000-resample bootstrap statistical analysis and publishing all raw run records and negative/inconclusive findings in `.execution/reports/T215.md`.

Out of scope (owner):
- V2 Linux x86_64 release packaging (T216); V3 stateful/I/O/fuzzing benchmark strata `W09`–`W11`, `W13` (V3 `T315`).

## Normative requirements
- Corpus prompts, comparable documentation allowances, starter files, hidden acceptance tests, model ID, sampling parameters, and budgets (3 repair attempts, 10 minutes/task) must be frozen and SHA-256 hashed before any measured run (DELIVERY_PLAN).
- Hidden acceptance tests never enter agent context; run order is randomized; all timeouts, crashes, and failed runs are retained in the raw dataset without selective omission (DELIVERY_PLAN, ACCEPTANCE_MATRIX).
- Improvement claims require a positive 95% bootstrap confidence interval lower bound for paired correctness difference and median cost/time `<= 2x` baseline; negative or inconclusive findings must be reported honestly and do not block technical release (DELIVERY_PLAN, RELEASE_CRITERIA).

## Acceptance criteria
Positive and boundary:
- [ ] All 24 V2 tasks (`W04`, `W07`, `W08` plus V1 regression classes) have frozen starter projects, prompts, hidden tests, and SHA-256 manifest hashes across Wolio V2, Go, Rust, and TypeScript
- [ ] All 24 × 10 × 4 paired runs complete (with timeouts/failures recorded) and raw JSONL transcripts are archived
- [ ] 10,000-resample paired bootstrap 95% confidence intervals and negative findings are published in `.execution/reports/T215.md`

Rejection and failure:
- [ ] Any leakage of hidden tests into agent context or selective omission of failed runs is rejected by benchmark verification checks

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written in `.execution/reports/T215.md`

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Evidence: `.execution/reports/T215.md` with frozen corpus SHA-256 manifest, raw run archive hashes, and 10,000-resample bootstrap tables.

## Dependencies and open items
- Depends on: T214; blocks: T216.
- Execution prerequisites: model API access and benchmark compute budget (record under `.execution/blockers/` if unavailable; never fabricate or simulate benchmark runs).
- Open: none.

