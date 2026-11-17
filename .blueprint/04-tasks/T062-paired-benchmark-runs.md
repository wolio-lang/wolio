`.blueprint/04-tasks/T062-paired-benchmark-runs.md`
# T062: Execute 10 paired benchmark runs per arm and archive raw records

| Field | Value |
|---|---|
| Milestone | `M06` |
| Priority | `P0` |
| Depends on | T061 |
| Blocks | T063 |
| Acceptance families | BENCH (complete runs including failures) |
| Bounded areas | `benchmarks/results` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T062.md` |

## Objective
Execute the 10-task × 5-paired-seed benchmark matrix across Wolio, Go, Rust and
TypeScript in randomized order (200 total runs) and archive every raw run record
without omission.

## Authority
- Normative: [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md) (Benchmark protocol), [RELEASE_CRITERIA.md](../05-quality/RELEASE_CRITERIA.md) (Evaluation claims), [BASELINE-V1.md](../BASELINE-V1.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `BENCH`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Taxonomy: [AI_BUG_TAXONOMY.md](../06-project-management/AI_BUG_TAXONOMY.md)

## Scope
In scope:
- Execution of 10 contract-first tasks × 5 paired runs per arm (Wolio, Go, Rust,
  TypeScript; 200 total runs; optional native Wolio stratum recorded separately if run).
- Complete raw JSONL/archive artifacts under `benchmarks/results` containing
  prompts, corpus hashes, model/tool versions, repair transcripts, diagnostics,
  hidden-test verdicts, token counts, wall time, cost and `AI_BUG_TAXONOMY`
  classifications.

Out of scope (owner):
- Paired bootstrap confidence intervals, economic threshold evaluation and
  final narrative report (T063).

## Normative requirements
- Run 10 tasks × 5 paired seeds/runs per arm across Wolio, Go, Rust and
  TypeScript in randomized order under the frozen T060/T061 protocol
  (DELIVERY_PLAN).
- Every timeout, tool error, repair exhaustion and failed acceptance test counts
  and must be retained in the raw dataset; selective omission or re-running
  failed seeds is forbidden (DELIVERY_PLAN, ACCEPTANCE_MATRIX,
  RELEASE_CRITERIA).
- Model API access and budget are mandatory execution prerequisites; missing
  access marks this task BLOCKED under `.execution/blockers/` and never permits
  simulated or fabricated benchmark runs (EXECUTION_CONTRACTS).

## Acceptance criteria
Positive and boundary:
- [ ] All 24 × 10 × 4 paired runs (960 primary runs) are executed and archived with verifiable SHA-256 digests
- [ ] Every record includes seed, run order, model/tool versions, corpus hash, repair attempts, tokens, wall-clock time, cost and hidden-test outcome

Rejection and failure:
- [ ] Timeouts, failed builds, failed repairs and escaped defects are retained in full with no omitted or replaced seeds
- [ ] Integrity validator rejects incomplete matrices, altered corpus hashes or unrecorded seeds

Completion:
- [ ] Common gates in EXECUTION_CONTRACTS pass
- [ ] No semantics beyond the cited authority; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Run the benchmark archive validator over `benchmarks/results` to verify 24 × 10
completeness per arm, hash consistency and failure retention.
Evidence: `.execution/reports/T062.md` with archive hashes and run-completion ledger.

## Dependencies and open items
- Depends on: T061; blocks: T063.
- Execution prerequisites: model API access, credentials and budget for 960
  paired agent runs (record a blocker under `.execution/blockers/` if
  unavailable).
- Open: none.

