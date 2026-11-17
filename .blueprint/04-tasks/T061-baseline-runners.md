`.blueprint/04-tasks/T061-baseline-runners.md`
# T061: Implement isolated Wolio, Go, Rust and TypeScript benchmark runners

| Field | Value |
|---|---|
| Milestone | `M06` |
| Priority | `P0` |
| Depends on | T060 |
| Blocks | T062 |
| Acceptance families | BENCH (equivalent capabilities and budgets) |
| Bounded areas | `benchmarks/runners` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T061.md` |

## Objective
Build the reproducible benchmark execution harness that runs Wolio, Go, Rust and
TypeScript arms under identical model settings, repair budgets, wall-clock
timeouts and hidden-test isolation.

## Authority
- Normative: [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md) (Benchmark protocol), [BASELINE-V1.md](../BASELINE-V1.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `BENCH`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Taxonomy: [AI_BUG_TAXONOMY.md](../06-project-management/AI_BUG_TAXONOMY.md)

## Scope
In scope:
- Runner adapters for Wolio (using the M04 local protocol and interpreter, plus
  an optional separate native stratum when M05 is available) and conventional
  Go, Rust and TypeScript baselines with their normal strict type checkers,
  linters, validation libraries and test runners.
- Enforcement of frozen model ID, sampling parameters, 3-repair-attempt budget,
  10-minute per-task timeout, randomized run order and isolated workspaces
  withholding hidden acceptance tests during agent turns.
- Structured telemetry capture (tokens, wall-clock time, cost, compiler/linter
  loops, repair attempts, hidden-test verdicts, `AI_BUG_TAXONOMY` tags).

Out of scope (owner):
- Executing the 10 × 5 paired benchmark campaign (T062); statistical
  bootstrap analysis and final evaluation report (T063).

## Normative requirements
- Benchmark against conventional languages (Go, Rust, TypeScript) configured
  with their normal strict compiler checks, linters, validation libraries and
  test tooling, not unvalidated strawmen (BASELINE-V1, DELIVERY_PLAN).
- Pin and record exact model identifier, sampling settings, tool versions,
  hardware description and pricing table before measured runs; enforce the same
  model/settings, comparable documentation allowance, 3 repair attempts and
  10-minute timeout per task across all arms (DELIVERY_PLAN).
- Keep hidden acceptance tests outside agent context during generation and
  repair; evaluate hidden acceptance tests only in a separate post-run harness
  step (DELIVERY_PLAN).

## Acceptance criteria
Positive and boundary:
- [ ] Runners for Wolio, Go, Rust and TypeScript execute end-to-end smoke checks against isolated workspaces and evaluate hidden tests post-run
- [ ] Exact tool versions, model configuration, hardware metadata and budget limits are validated and recorded in run headers
- [ ] Run-order randomization is deterministic from a recorded seed

Rejection and failure:
- [ ] Exceeding 3 repair attempts or the 10-minute wall-clock limit terminates the run and records a budget/timeout failure without dropping the record
- [ ] Mismatched corpus hashes or unpinned tool/model settings abort runner startup

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
Run deterministic dry-run/smoke harness checks verifying budget enforcement,
timeout handling, hidden-test isolation and schema completeness across all four
language runners.
Evidence: `.execution/reports/T061.md` with runner configurations and pinned baseline tool versions.

## Dependencies and open items
- Depends on: T060; blocks: T062.
- Execution prerequisites: Go, Rust and TypeScript (Node/tsc) baseline toolchains
  installed on the host.
- Open: none.

