`.blueprint/04-tasks/T063-benchmark-analysis-report.md`
# T063: Compute paired bootstrap intervals and publish the M06 evaluation report

| Field | Value |
|---|---|
| Milestone | `M06` |
| Priority | `P0` |
| Depends on | T062 |
| Blocks | none |
| Acceptance families | BENCH (intervals, negative findings); completes M06 |
| Bounded areas | `benchmarks/analysis`, evaluation documentation |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T063.md` |

## Objective
Analyze the archived M06 paired benchmark dataset using 10,000-resample paired
bootstrap 95% confidence intervals and publish the reproducible evaluation
report, including negative or inconclusive findings, completing M06.

## Authority
- Normative: [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md) (Benchmark protocol), [RELEASE_CRITERIA.md](../05-quality/RELEASE_CRITERIA.md) (Evaluation claims), [BASELINE-V1.md](../BASELINE-V1.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `BENCH`, M06 row of [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md), [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Taxonomy: [AI_BUG_TAXONOMY.md](../06-project-management/AI_BUG_TAXONOMY.md)

## Scope
In scope:
- Deterministic analysis tool computing task success proportion, escaped
  defects, regressions, repair iterations, compiler feedback loops, token usage,
  wall-clock time, cost and `AI_BUG_TAXONOMY` breakdowns from T062 raw records.
- Paired bootstrap 95% confidence intervals over tasks (10,000 resamples with
  fixed seed) comparing Wolio against Go, Rust and TypeScript.
- Published M06 evaluation report documenting methodology, raw data links,
  confidence intervals, economic tradeoffs, training-familiarity limitations and
  negative or inconclusive findings.

Out of scope (owner):
- Post-V1 language extension admissions (governed separately by `POST_V1_PLAN`).

## Normative requirements
- Report success proportion, escaped defects, regressions, repairs, tokens,
  wall-clock time and cost, with paired bootstrap 95% intervals over tasks
  using 10,000 resamples (DELIVERY_PLAN).
- An improvement claim is permitted only when the paired correctness difference
  has a positive 95% bootstrap lower bound and median cost and wall-clock time
  are no more than 2× the baseline; otherwise the report must state tradeoffs,
  negative results or inconclusive evidence (DELIVERY_PLAN, RELEASE_CRITERIA).
- Negative or inconclusive benchmark findings do not block a technical release,
  must never be hidden, and must not extrapolate to unsupported stateful or
  concurrent domains or all AI models (BASELINE-V1, DELIVERY_PLAN,
  RELEASE_CRITERIA).

## Acceptance criteria
Positive and boundary:
- [ ] Re-running the analysis command over `benchmarks/results` reproduces identical tables and 10,000-resample bootstrap 95% intervals
- [ ] Report includes all 24 tasks, all four arms, `AI_BUG_TAXONOMY` counts, cost/time ratios and training-familiarity limitations

Rejection and failure:
- [ ] Any improvement claim failing the positive 95% lower-bound or <= 2× median cost/time threshold is rejected by the report check
- [ ] Negative findings, regressions and tasks where baselines outperform Wolio are reported prominently without omission

Completion:
- [ ] All M06 DELIVERY_PLAN and ACCEPTANCE_MATRIX `BENCH` gates pass
- [ ] Common gates in EXECUTION_CONTRACTS pass; no unrelated changes
- [ ] Report written under the EXECUTION_CONTRACTS report contract

## Verification
```bash
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace
```
Execute the reproducible benchmark analysis command over `benchmarks/results`
and verify bit-for-bit reproduction of the summary tables and confidence
intervals.
Evidence: `.execution/reports/T063.md` with analysis command, seed, interval tables and published report path.

## Dependencies and open items
- Depends on: T062; blocks: none.
- Open: none.

