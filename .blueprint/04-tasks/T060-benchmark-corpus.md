`.blueprint/04-tasks/T060-benchmark-corpus.md`
# T060: Freeze 10-task contract & protocol benchmark corpus, prompts and hidden tests

| Field | Value |
|---|---|
| Milestone | `M06` |
| Priority | `P0` |
| Depends on | T044 |
| Blocks | T061 |
| Acceptance families | BENCH (corpus and 10-task eligibility) |
| Bounded areas | `benchmarks/corpus` |
| Execution status | `.execution/task-status.md` (sole source) |
| Report | `.execution/reports/T060.md` |

## Objective
Freeze the 10-task contract-and-protocol benchmark corpus, prompts, starter
workspaces and independent hidden acceptance suites across Wolio, Go, Rust and
TypeScript before any measured M06 evaluation run. (The full 24-task algorithmic
comparative matrix is scheduled for V2 `T215`).

## Authority
- Normative: [DELIVERY_PLAN.md](../06-project-management/DELIVERY_PLAN.md) (Benchmark protocol), [BASELINE-V1.md](../BASELINE-V1.md)
- Acceptance: [ACCEPTANCE_MATRIX.md](../05-quality/ACCEPTANCE_MATRIX.md) `BENCH`, [DEFINITION_OF_DONE.md](../05-quality/DEFINITION_OF_DONE.md)
- Taxonomy: [AI_BUG_TAXONOMY.md](../06-project-management/AI_BUG_TAXONOMY.md) (`W01`–`W16`)

## Scope
In scope:
- 10 frozen contract-first benchmark tasks covering specification adherence,
  contract verification (`requires`/`ensures`), error propagation (`Result<T, E>`),
  anti-workaround resistance (`WOLIO-ANALYZER-001`..`005`), and agent repair
  cycles within V1 capabilities.
- Equivalent starter projects, comparable language documentation allowances and
  task prompts for Wolio, Go, Rust and TypeScript.
- Independent hidden acceptance test suites for each task and arm, plus a
  manifest of SHA-256 hashes over all corpus inputs.

Out of scope (owner):
- Baseline language runner execution harnesses (T061); executing the paired
  runs per arm (T062); bootstrap statistical analysis and report (T063).
- Algorithmic loops, collections, and decimal benchmarks (scheduled for V2 `T215`).

## Normative requirements
- V1 M06 evaluation uses exactly 10 tasks focused on contract checking, error
  handling, and anti-workaround defenses across Wolio, Go, Rust and TypeScript; tasks
  requiring loops, collections, concurrency, databases, or external I/O are excluded
  until V2/V3 (DELIVERY_PLAN).
- Corpus tasks, prompts, comparable documentation allowances, hidden independent
  acceptance tests and budgets (3 repair attempts, 10 minutes/task) are frozen
  and hashed before any measured run (DELIVERY_PLAN, ACCEPTANCE_MATRIX).
- Hidden acceptance tests must never appear in agent context or starter
  workspaces (DELIVERY_PLAN).

## Acceptance criteria
Positive and boundary:
- [ ] All 10 tasks have starter workspaces, prompts and hidden acceptance tests for Wolio, Go, Rust and TypeScript
- [ ] Every task uses only capabilities supported equivalently in V1 Wolio and the three baseline languages
- [ ] Corpus manifest records deterministic SHA-256 hashes for all prompts, starter files and hidden tests

Rejection and failure:
- [ ] Any task requiring unsupported loops, collections, concurrency, database, network or secret features is rejected by corpus eligibility checks
- [ ] Any leakage of hidden acceptance test files into agent-visible workspace paths is rejected by verification checks

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
Verify corpus eligibility, hidden-test isolation and SHA-256 manifest hashes for
all 24 tasks across all four language arms.
Evidence: `.execution/reports/T060.md` with the 24-task inventory and frozen corpus hash manifest.

## Dependencies and open items
- Depends on: T044; blocks: T061.
- Open: none.

